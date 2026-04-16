## re-family

> Use this file together with [`AGENTS.md`](./AGENTS.md).

# CLAUDE.md

Use this file together with [`AGENTS.md`](./AGENTS.md).

## Source Of Truth

- Business requirements, domain rules, and product behavior live in [`doc/`](./doc/).
- Repository structure lives in [`ARCHITECTURE.md`](./ARCHITECTURE.md).
- Frontend implementation rules live in:
  - [`docs/FRONTEND-SYSTEM.md`](./docs/FRONTEND-SYSTEM.md)
  - [`docs/FRONTEND-CHECKLIST.md`](./docs/FRONTEND-CHECKLIST.md)
  - [`docs/FRONTEND-EXAMPLES.md`](./docs/FRONTEND-EXAMPLES.md)
- Deployment notes live in [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md).

## Current Repository Shape

```text
re-family/
├── apps/
│   ├── api/        # NestJS 11 + Fastify API
│   └── web/        # Next.js 15 App Router frontend
├── packages/
│   ├── config/
│   ├── types/
│   ├── ui/
│   └── validators/
├── infrastructure/
│   ├── docker/
│   ├── helm/
│   ├── scripts/
│   └── terraform/
├── docs/
└── doc/
```

## Practical Defaults

- API prefix: `/api/v1`
- Swagger: `/api/docs`
- Local API default: `http://localhost:3001`
- Local web default: `http://localhost:3000`
- Frontend routes are split across `(auth)`, `(family)`, `(advisor)`, `(admin)`, plus public pages

## Application Boundaries

### `apps/web`

- owns routes, page composition, domain UI, API wiring, and portal shells
- uses `@re-family/ui/<component>` subpath imports for shared primitives
- keeps business logic out of `packages/ui`

### `apps/api`

- owns backend modules and cross-cutting runtime concerns
- `CoreModule` wires config, database, tenancy, auth, permissions, mail, and storage
- `SharedModule` exposes health/GDPR style shared endpoints

### `packages/ui`

- generic design-system components only
- no app-specific business logic

## Commands

Workspace-level:

```bash
pnpm dev
pnpm build
pnpm lint
pnpm check-types
pnpm test
```

Useful scoped commands:

```bash
pnpm --filter @re-family/web check-types
pnpm --filter @re-family/api test
```

## Frontend Reminder

Before changing frontend code:

1. check the requirement in `doc/`
2. read `docs/FRONTEND-SYSTEM.md`
3. decide whether the change belongs in `apps/web` or `packages/ui`
4. verify with:

```bash
corepack pnpm --filter @re-family/web exec tsc --noEmit
git diff --check
```

## Documentation Hygiene

When setup, workflow, or architecture changes:

- update the relevant file in `docs/` or the repository root
- do not treat `doc/` as a place for implementation notes unless business requirements changed
- remove stale duplicate docs instead of keeping conflicting versions around

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vpavlov-me) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
