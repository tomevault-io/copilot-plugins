## robo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workspace Layout

`C:\htdocs\robo` is a **multi-repo workspace**, not a monorepo. There is no root `.git` — each service folder is its own independent git repository with its own `package.json`, dependencies, CI, and Changesets versioning.

| Folder | Type | Port | Stack | Role |
|---|---|---|---|---|
| `ms-robo-admin/` | NestJS backend | 3002 | NestJS 11 + CQRS + TypeORM + PostgreSQL 17 | Admin API. **Sole schema owner.** |
| `ms-robo-web/` | NestJS backend | 3001 | Same as above | Client API. **Schema consumer** — never creates migrations. |
| `mfe-robo-admin/` | React frontend | 5173 | React 19 + Vite 7 + Tailwind 4 + HeroUI + Zustand + TanStack Query | Admin dashboard. Talks to `ms-robo-admin`. |
| `mfe-robo-web/` | React frontend | 5174 | Same as above | Client portal. Talks to `ms-robo-web`. |
| `ms-robo-dataloader/` | Placeholder | — | — | Empty repo (only `.git`, `.gitignore`, README). Do not put work here without confirming with the user. |
| `documents/` | Spec docs | — | — | Source-of-truth handover specs (`01-project-overview.md` … `07-business-logic-qa.md`). Read these before touching unfamiliar areas. |
| `tasks/feat/` | Feature work | — | — | Each numbered subfolder (e.g. `01_portfolio_export`, `02_datalake_modify`) contains spec / design / Q&A / `client-reply.md` files for one feature. See "Feature task convention" below. |
| `backup/` | Snapshots | — | — | Schema dumps, view exports. Read-only reference. |
| `infra/` | Infra config | — | — | `docker-compose.infra.yaml` (currently empty). |

**Implication**: When working in any service, `cd` into that folder before running commands. `package.json` scripts only exist inside service folders, never at the root.

## Common commands

All backend services use **Bun** (`>=1.0.0`); both frontends use **Node 22**. Run from the relevant service directory.

### Backend (`ms-robo-admin`, `ms-robo-web`)

```bash
bun install                      # install deps
bun run start:dev                # dev with watch (port 3002 / 3001)
bun run build                    # nest build → dist/
bun run start:prod               # run dist/main
bun run lint                     # eslint
bun run format                   # prettier
bun run test                     # jest (uses @swc/jest in ms-robo-admin, ts-jest in ms-robo-web)
bun run test:watch
bun run test:cov                 # coverage → ./coverage
bun run test:e2e                 # spins up docker-compose.e2e.yaml first
bun run test -- path/to/file.spec.ts            # run a single test file
bun run test -- -t "should do X"                # run tests matching name
```

### Backend DB (only in `ms-robo-admin`, the schema owner)

```bash
bun run db:generate              # generate a new migration from entity diff
bun run db:create                # create empty migration
bun run db:migrate               # run pending migrations
bun run db:revert                # revert last migration
bun run schema:drop              # drop all tables (destructive)
bun run seed:run                 # run seeders
bun run sync:sus8 -- --limit 10  # SUS8 datalake sync (one-shot CLI)
bun run db:sync                  # push entity files into ms-robo-web (see "Schema ownership")
bun run db:sync:check            # diff entities between admin and web (read-only)
```

### Frontend (`mfe-robo-admin`, `mfe-robo-web`)

```bash
bun install                      # bun works fine even though node is the runtime
bun run dev                      # vite dev (port 5173 / 5174)
bun run build                    # vite build (mfe-robo-web also runs tsc first)
bun run typecheck                # tsc --noEmit
bun run lint                     # eslint .
bun run lint:fix
bun run format                   # prettier
bun run knip                     # detect unused exports/files
bun run test                     # vitest run
bun run test:watch
bun run test:ui                  # vitest UI
bun run test path/to/file.test.tsx              # run a single test file
bun run test -- -t "label"                      # run tests matching name
```

## Schema ownership (critical rule)

`ms-robo-admin` is the **sole owner** of the database schema. All `@Entity` definitions, migrations, and seeds live there. `ms-robo-web` is a **consumer** — its `entities/` directory is a synced copy that should never be hand-edited and never has its own migrations.

To propagate a schema change:

1. Add/edit the entity in `ms-robo-admin/src/infrastructure/persistence/entities/...`.
2. `cd ms-robo-admin && bun run db:generate` to create the migration.
3. `bun run db:migrate` against the local DB.
4. `bun run db:sync` to push the updated entity files into `ms-robo-web/`.
5. Verify with `bun run db:sync:check`.

If `ms-robo-web` shows entity drift, run `db:sync:check` first — never patch `ms-robo-web` entities directly.

## Backend architecture (both `ms-robo-*` services)

Clean Architecture + CQRS, with the same layout in both backends:

```
src/
├── core/
│   ├── domain/{context}/         # Entities, value objects, domain errors
│   └── application/{context}/    # Commands, queries, handlers, ports (interfaces)
├── infrastructure/
│   ├── persistence/              # TypeORM entities, repositories, migrations, mappers
│   ├── external/                 # Adapters: Keycloak, FNZ/Tapico, SUS8 Redshift, Onboarding API, JWT
│   ├── adapters/                 # Other infra adapters
│   ├── config/                   # Config modules per integration
│   └── logger/                   # Pino
├── presentation/
│   ├── http/{context}/           # Controllers, DTOs, guards, decorators
│   └── modules/                  # NestJS feature modules (DI wiring)
├── shared/                       # Symbol tokens, shared DTOs, utils
└── types/
```

**Path aliases** (TS + Jest both): `@core/*`, `@infrastructure/*`, `@persistence/*` (= `infrastructure/persistence`), `@presentation/*`, `@shared/*`, `@test/*`. Use these in imports — never use deep relative paths like `../../../core/...`.

**Dependency injection** uses **Symbol tokens**, not class references. When wiring a port to its adapter:

```typescript
export const USER_REPOSITORY = Symbol('USER_REPOSITORY');
// in module providers:
{ provide: USER_REPOSITORY, useExisting: UserRepositoryAdapter }
```

When adding a new use case, follow the existing CQRS layout: `core/application/{context}/commands/{name}/{name}.command.ts` + `.handler.ts`, with the port living under `core/application/{context}/ports/`. Look at an existing slice (e.g. `core/application/portfolio/queries/`) before inventing a new layout.

**`ms-robo-admin` domain contexts** (13): identity, authorization, account, client, portfolio, portfolio-request, portfolio-template, questionnaire, funding-request, withdraw-request, trading, instrument, user-sync. The full list with entity ER summary lives in `documents/02-spec-ms-robo-admin.md`.

## Frontend architecture (both `mfe-robo-*` apps)

```
src/
├── App.tsx              # Router config — all routes defined here
├── main.tsx
├── core/                # Shared infra: apiManager (axios + token refresh), auth store/hooks, layouts, shared components
├── features/{name}/     # One folder per feature
│   ├── apis/            # API service functions (axios calls)
│   ├── hooks/           # React Query queries/mutations
│   ├── components/      # Feature-specific components
│   ├── pages/           # Page components
│   └── types/           # TS interfaces
├── constants/           # Routes, status colors
└── styles/
```

When adding a feature, follow the `features/{name}/{apis,hooks,components,pages,types}` shape exactly — there are 16 examples in `mfe-robo-admin/src/features/`.

State management split:
- **Server state** → TanStack Query (React Query) hooks under `features/*/hooks/`
- **Client/UI state** → Zustand stores under `core/auth/` or feature-local stores
- **Forms** → React Hook Form + Zod schemas

The two frontends do **not** share code at the file level — they are separate repos. If a component needs to be shared, copy it explicitly and document it.

## Authentication

Two auth flows on each frontend, both ending in JWT tokens stored in `localStorage`:

1. **Username/password** → `POST /v1/auth/login`
2. **Keycloak SSO (PKCE)** — frontend generates `codeVerifier`/`codeChallenge`, redirects to Keycloak, receives code at `/auth/callback`, exchanges via backend.

Token storage uses prefixes to keep the two portals isolated on the same domain:
- Admin UI: `admin-accessToken`, `admin-refreshToken`
- Client UI: `portal-accessToken`, `portal-refreshToken`

Backend guards: `JwtAuthGuard` is global (use `@Public()` to opt out). Add `@Roles(RoleEnum.Admin)` and `@RequirePermission('resource:action')` for finer-grained checks. Permission scopes are `global > company > branch > own`.

## Local infrastructure

Required services run via Docker Compose (PostgreSQL on **5433**, Keycloak on 8080, SonarQube on 9100). The `infra/docker-compose.infra.yaml` file at the workspace root is currently empty — each service ships its own compose file, and the canonical infra spec lives in `documents/01-project-overview.md` §7.1. Confirm the right compose file with the user before standing services up.

DB name: `uob_portal`. Test users for Keycloak (System Admin / Client / TR / Back Office / Staff) are listed in `documents/01-project-overview.md` §7.2.

## Feature task convention (`tasks/feat/`)

Feature work is tracked in numbered subfolders under `tasks/feat/`. Each folder is one client-facing feature and follows a standard layout when complete:

- `README.md` — client entry point with reading order
- `01-approach.md`, `02-high-level-design.md`, `03-dataflow.md`, `04-mapping.md` — client-facing layered docs
- `client-reply.md` — Yes/No decision form for client confirmation (see `tasks/feat/01_portfolio_export/client-reply.md` for the canonical format)
- `spec-*.md`, `design-*.md`, `qa-*.md` — internal dev-only deep references
- `user_prompt_*.txt` — conversation snapshots from the planning session

When asked to plan a new feature, follow the format used by `02_datalake_modify`. The split between client-facing docs (numbered 01-04 + README + client-reply) and dev-only references (`spec-*`, `design-*`, `qa-*`) is intentional — do not collapse them.

## Git & commits

- **Conventional Commits enforced** by Husky + Commitlint in every service. Format: `type(scope): subject` (e.g. `feat(portfolio): add export endpoint`).
- **Do NOT add `Co-Authored-By: Claude` (or any `Co-Authored-By:`) trailers** — commitlint rejects them and the commit will fail. This is a hard constraint, not a preference.
- **Versioning**: Changesets. Run `bun run changeset` to add a changelog entry, then `bun run version:bump` to roll versions and propagate to deployment manifests via `scripts/sync-version.sh`.
- Each service is a separate git repo, so `git status` / `git commit` only see one service at a time. Always `cd` into the service first.

## Where to read more

The richest context lives in `documents/` (created during handover, kept as the source of truth):

- `01-project-overview.md` — system architecture, ports, stack, RBAC, integrations, deployment
- `02-spec-ms-robo-admin.md` — backend admin details (52 entities, ER diagram, all controllers, 13 domains)
- `03-spec-ms-robo-web.md` — backend client details
- `04-spec-mfe-robo-admin.md` — admin frontend (16 features, routes, permissions)
- `05-spec-mfe-robo-web.md` — client portal frontend
- `06-spec-infra-dataloader.md` — original intent for the empty `ms-robo-dataloader` repo
- `07-business-logic-qa.md` — business logic answers
- `handover_extracted.txt` — full text of the original handover PDF

Read the relevant spec doc before making non-trivial changes — they describe intent that the code alone doesn't reveal.

---
> Source: [mrleloi/robo](https://github.com/mrleloi/robo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
