## brew-lab

> A web application for tracking and analyzing coffee brewing experiments.

# Brew Lab

A web application for tracking and analyzing coffee brewing experiments.

Read `specs/index.md` for project overview and how the spec/conventions structure fits together.

## Three sources of truth (in order)

1. **Code** — authoritative for behavior, fields, URLs, layouts. When something disagrees with the code, the code wins.
2. **`conventions/`** — *prescriptive* cross-cutting rules new code should follow. Authoritative for new work in those areas.
3. **`specs/`** — *reference context* for each feature: intent, design decisions, expected user workflows, and pointers into the code. **Not authoritative** — read for the *why*, then trust the code for the *what*.

## Conventions to follow when writing new code

Read the relevant file before adding code in these areas:

| When you're doing this | Read |
| --- | --- |
| Adding/modifying an HTTP endpoint | `conventions/api-conventions.md` |
| Adding a migration or DB pattern | `conventions/database-conventions.md` |
| Adding a UI component / color / spacing | `conventions/design-system.md` |
| Touching service worker / manifest / push | `conventions/pwa.md` |
| Changing VPS / Docker / CI | `conventions/deployment.md` |

These are authoritative — match them in new code.

## How to start work on a feature

1. Identify which feature(s) the task touches.
2. Read the **code** along the relevant path — it's the source of truth.
3. Skim `specs/<feature>.md` for the *why* (design decisions, intent, expected workflow). Use its **Code Pointers** section to navigate.
4. If the spec's `last_updated` is months old, treat its specific claims as hints and verify against the code before relying on them.
5. Check `specs/index.md` for the dependency graph if cross-feature work is involved.
6. If you're adding code in a cross-cutting area (HTTP, DB, UI, PWA, deployment), read the matching `conventions/` file first.

## Specs as reference (not source of truth)

- Code is authoritative. Specs encode intent, decisions, and workflows that aren't obvious from the code.
- When spec and code disagree, treat the **code** as authoritative; the spec is likely stale.
- Update a spec **when your change shifts intent, makes a new design decision, or alters a user-visible workflow**. Pure refactors, bug fixes, and renames need no spec touch.
- In plan mode, if your change touches a feature with a spec and shifts intent or workflow, mention the spec edit alongside the code change. Otherwise omit.
- Each spec carries `created` + `last_updated` dates. Bump `last_updated` when you edit a spec; don't touch `created`.
- `status: needs-review` in frontmatter is an explicit flag from a prior author — surface it before relying on that spec, but it doesn't block the work.

## Running the App

### Database (from project root)
```bash
docker compose up -d   # Start PostgreSQL
docker compose down    # Stop PostgreSQL
```

### Backend (from `backend/` directory)
```bash
make run        # Start backend server (auto-runs migrations on startup)
```
The Makefile auto-loads `backend/.env`, so no manual env setup is needed.

### Frontend (from `frontend/` directory)
```bash
npm install     # First time only
npm run dev     # Start dev server
```

## Backend Commands (Make Targets)

| Command | Description |
|---------|-------------|
| `make build` | Build all binaries |
| `make run` | Run server (auto-runs migrations) |
| `make tidy` | go mod tidy |
| `make test` | Run all Go tests |
| `make seed-user EMAIL=...` | Create admin user (no password — auth is magic-link only) |

## Frontend Commands (npm scripts)

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Vite dev server |
| `npm run build` | Production build (`tsc -b && vite build`) |
| `npm run test` | Run vitest tests once |
| `npm run test:watch` | Run vitest in watch mode |
| `npm run preview` | Preview build |
| `npm run lint` | Run ESLint |

## Verifying Changes

Quick verification steps for agents after making changes:

| Change Type | Verification |
|-------------|--------------|
| Backend code | `cd backend && go build ./...` |
| Frontend code | `cd frontend && npm run lint && npm run build && npm run test` |
| Migrations | Automatic on server start (`make run`) |

**Note:** `npm run build` runs `tsc -b` which type-checks test files strictly (unlike `tsc --noEmit`). Always run the full build to catch type errors in tests — this matches CI.

## Conventions Worth Knowing

These aren't obvious from a glance at the code — calling them out saves a search.

- **Domain layout (backend)**: each domain lives in `backend/internal/domain/<x>/` with the same file shape: `entity.go` (types), `repository.go` (interface), `repository_pg.go` (Postgres impl), `handler.go` (HTTP), `errors.go` (typed errors when needed). Keep new domains consistent with this.
- **Migrations**: sequentially numbered SQL pairs in `backend/internal/database/migrations/NNN_<name>.{up,down}.sql`. They run automatically on server start.
- **Shared types**: backend types live in `backend/internal/domain/<x>/entity.go`; frontend types live in `frontend/src/api/<x>.ts` next to the API client. There is no monorepo — keep them in sync manually.
- **Nullable fields**: Go uses `*string` / `*int`; TypeScript uses `string | null` / `number | null`.
- **Anti-enumeration**: auth endpoints (`/auth/magic-link`, `/auth/verify-magic-link`, `/auth/verify-friend-magic-link`) always return 200 with a generic message regardless of whether the email matched. Don't break this when extending.
- **Postgres unique-violation mapping**: handlers detect `*pgconn.PgError` with `Code == "23505"` and inspect `ConstraintName` to convert violations into typed errors (e.g., `friend.ErrDuplicateEmail` → 409). When adding a constraint, give it a meaningful name and wire the mapping if the handler should differentiate.
- **Frontend test fixtures**: shared-type mocks in tests should use the factories in `frontend/src/test/fixtures.ts` (`makeFriend`, `makeBrew`, `makeCoffee`, etc.) rather than inline objects. Adding a field to a shared type then only touches `fixtures.ts`.

## File Organization
```
specs/           # Feature reference context (the why)
conventions/     # Prescriptive cross-cutting rules for new code
backend/         # Go backend (chi router)
frontend/        # React + TypeScript (Vite)
docker-compose.yml
```

---
> Source: [poimgs/brew-lab](https://github.com/poimgs/brew-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
