## project-architecture

> Core project architecture — bounded contexts, layer rules, cross-context contracts, infrastructure constraints


# Project Architecture

Modular monolith in production. Nine bounded contexts. Hardening for reliability, auditability, and scalability is ongoing — rules in this file are the current baseline, not a ceiling.

## Backend root modules

| File | Role |
|------|------|
| `main.py` | App composition root — creates FastAPI app, mounts middleware and exception handlers |
| `routes.py` | Router aggregation — imports all context routers and registers them under `/api` |
| `startup.py` | Lifespan context manager — DB/Redis init, warm-up, demo seeding, shutdown |
| `scheduler.py` | Background scheduled tasks — Xero nightly sync loop; add new recurring jobs here |

## Bounded contexts

`identity`, `operations`, `finance`, `catalog`, `purchasing`, `jobs`, `documents`, `reports`, `assistant`

Each context owns: `domain/`, `application/`, `infrastructure/`, `api/`, and optionally `ports/`.

## Layer dependency rules (strict)

```
shared/kernel/              ← pure domain primitives, zero infra deps
shared/infrastructure/      ← cross-cutting infra (db, redis, config, middleware)
{context}/domain/           ← imports only shared/kernel + stdlib + pydantic
{context}/ports/            ← imports only domain + shared/kernel
{context}/infrastructure/   ← implements ports, imports domain + shared/infrastructure
{context}/application/      ← orchestrates domain + infra, imports other contexts' application facades
{context}/api/              ← thin HTTP transport, imports only own application layer + shared/api
```

Composition roots (`main.py`, `routes.py`, `startup.py`, `scheduler.py`) are exempt from cross-context import rules — their purpose is to wire the whole system together.

### What each layer may contain

| Layer | Allowed | Forbidden |
|-------|---------|-----------|
| `domain/` | Entities, value objects, enums, state machines, pure computations, domain errors, typed DTOs/commands/results | IO, imports from infrastructure, imports from application, imports from api, dicts as primary data contract |
| `ports/` | Protocol definitions using domain types only | `conn` parameters, `dict` return types on mutations, infrastructure types |
| `infrastructure/` | Repos (CRUD + queries), schema, adapters | Business logic, validation beyond data integrity, cross-context table access, orchestration |
| `application/` | Use-case orchestration, transaction boundaries, cross-context facade calls, event emission | FastAPI types, raw SQL, direct `get_connection()` outside transaction scope, mutable global state |
| `api/` | Route handlers, request/response mapping, FastAPI deps, error translation | Business logic, multi-step orchestration, raw SQL, direct repo imports |

### Cross-context access

```python
# FORBIDDEN — direct infra import across contexts
from operations.infrastructure.withdrawal_repo import withdrawal_repo

# FORBIDDEN — raw SQL against another context's tables
await conn.execute("UPDATE withdrawals SET payment_status = 'paid' ...")

# CORRECT — application-layer facade
from operations.application.withdrawal_facade import link_to_invoice
```

Any cross-context mutation must go through the owning context's application facade. The owning context controls its own state transitions.


## Database

Postgres everywhere — dev, test, production. No SQLite. Local dev uses the local Supabase stack. Tests run against the local Supabase Postgres database after `supabase db reset --local`.

All SQL uses native Postgres syntax (`$1`, `$2` placeholders, `NOW()`, `ON CONFLICT`, `::date` casts). There is no dialect abstraction layer.

## Transaction management

The DB layer uses a contextvar-based Unit of Work:
- `transaction()` stores the transactional connection in `_tx_conn` contextvar
- `get_connection()` returns the ambient transactional connection when inside a `transaction()` block, otherwise returns the pool proxy
- Repos never accept `conn` parameters — they call `get_connection()` which automatically participates in the ambient transaction
- Application-layer use cases own the transaction boundary via `async with transaction():`

Do not add `conn` parameters to repos or port signatures. Do not call `get_connection()` directly outside a `transaction()` block in application code.

## Shared package

`shared/kernel/` contains: Entity, AuditedEntity, CurrentUser, LineItem, Address, DomainError, Event, round_money. Pure domain primitives with zero infrastructure deps.

`shared/infrastructure/` contains: config, db, redis, event_hub, logging, metrics, middleware. These import only from `shared/kernel/` and nothing from any bounded context.

`shared/api/` contains: `deps.py` (auth DI), `auth_deps.py` (JWT validation + `CurrentUser` construction), `auth_provider.py` (provider-agnostic claim extraction), `websocket.py` (domain event WebSocket router), `health.py` (liveness/readiness probes). These are cross-cutting transport concerns, not owned by any single bounded context.

## Auth model

JWT validation is stateless — no DB roundtrip per request. `shared/api/auth_deps.py` decodes the token and builds `CurrentUser`. Claim extraction is delegated to `shared/api/auth_provider.py`. Supabase is the auth provider in every environment.

To support a new auth provider: add a branch in `auth_provider.py`. No other files change.

`CurrentUser.organization_id` is the org isolation key. Every write must be scoped to it. In production, tokens without `organization_id` are rejected at the transport layer. See `deployment-hardening.mdc` for the full auth hardening spec.

## Schema and migrations

- Database schema changes land as SQL migrations under `supabase/migrations/`.
- The local source of truth is the checked-in Supabase project, not backend Python DDL files.
- Apply local schema changes with `supabase db reset --local` through `pixi run tests backend`, `pixi run tests`, or `pixi run supabase reset`.

## File size

Keep files under 300 lines. Split by responsibility (not by endpoint or alphabetical grouping).

## Monorepo layout

```
sku-ops/
├── backend/          # deployable app + its tests + Python packaging
│   ├── pyproject.toml # production + dev deps, ruff, pytest, commitizen
│   ├── uv.lock
│   └── tests/        # backend unit/integration/api tests
├── frontend/         # React SPA (co-located unit tests in src/__tests__/)
├── e2e/              # cross-stack Playwright tests
├── devtools/         # seeds, evals, scripts — not deployed
└── pixi.toml         # unified dev task runner
```

Each deployable owns its tests. Devtools live at the workspace root. All Python dependencies and tool config live in `backend/pyproject.toml` (production under `[project.dependencies]`, tooling under `[dependency-groups].dev`). The Docker image excludes `backend/tests/` via `.dockerignore` and skips dev deps via `uv sync --no-dev --directory backend`.

## Dependency hygiene

- Never install packages speculatively. Only add what the current feature requires.
- Before creating anything, check what already exists.
- Python deps go in `backend/pyproject.toml` only: production in `[project.dependencies]`, tooling (pytest, ruff, commitizen, etc.) in `[dependency-groups].dev`.

## Commit conventions

Commitizen with gitmoji conventional commits: `type(scope): message`.

## Branch workflow

```
main <- production (auto-deploys, protected)
 ^ PR
dev <- integration branch
 ^ PR
feature/* | fix/* <- working branches
```

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
