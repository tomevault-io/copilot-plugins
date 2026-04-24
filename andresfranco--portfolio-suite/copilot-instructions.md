## portfolio-suite

> You are working in a full-stack repo with:

You are working in a full-stack repo with:
- DB: PostgreSQL (async via asyncpg) using SQLAlchemy 2.x + Alembic migrations.
- Backend: FastAPI (Pydantic v2, async endpoints).
- Frontend: React 19 + TypeScript, Vite, TanStack Query, React Router 6+, React Hook Form + Zod.
- Tests: pytest + httpx AsyncClient + Testcontainers (Postgres); Vitest + RTL + MSW; Playwright for e2e.

# Golden principles
- Optimize for **scalability, maintainability, and testability** over cleverness.
- Favor functional, declarative code; use classes where idiomatic (ORM models, Pydantic schemas, exceptions).
- Clear boundaries: **domain** (pure logic) vs **infrastructure** (web/DB).
- Async end-to-end for I/O; never block the event loop.
- Small, composable modules; single responsibility; DRY.
- Use descriptive names (e.g., `is_active`, `has_permission`).
- “Receive an Object, Return an Object” (RORO) at API/service boundaries.
- Every change ships with tests and docstrings (and README notes when architectural).

# Backend (FastAPI) rules
- Project layout:
  /app
    /api (routers grouped by feature; version under /api/v1)
    /core (config, security, logging, lifespan)
    /db (models, database.py, repositories, uow.py, migrations/)
    /domain (entities, services/usecases)
    /schemas (Pydantic request/response; no DB concerns)
    /workers (background tasks if needed)
- App factory + lifespan context manager (prefer over @app.on_event).
- Config via pydantic-settings; 12-factor envs (`DATABASE_URL`, `ENV`, `SECRET_KEY`).
- Data layer:
  - SQLAlchemy 2.x declarative models; snake_case tables; `created_at/updated_at` with server defaults.
  - Async engine + sessionmaker; one session per request (DI dependency).
  - **Repository + Service (Use Case) pattern**; no ORM calls from routers.
  - **Unit of Work** for transactional operations; commit at use-case boundary.
  - Prevent N+1 with `selectinload`/`joinedload`; add appropriate indexes.
  - Prefer Postgres-native types (UUID, JSONB, CITEXT) where useful.
- Validation/contracts:
  - Pydantic v2 models only at edges (I/O). Keep domain types framework-agnostic.
  - Typed responses; consistent pagination (`limit/offset` with caps or keyset).
- Migrations: Alembic `autogenerate` then hand-tune; never bypass migrations.
- Errors:
  - Expected errors → `HTTPException` with consistent error envelope `{code, message, details?}`.
  - Unexpected errors → middleware; structured logs; no secrets in logs.
- Security:
  - OAuth2 Bearer JWT (access/refresh); password hashing via argon2/bcrypt.
  - CORS restricted to allowed origins; secure headers; input validation everywhere.
  - Principle of least privilege; parameterized queries only.
- Observability:
  - JSON structured logging, correlation/request IDs.
  - `/healthz` and `/readyz`; OpenTelemetry traces (optional); Prometheus metrics (optional).
- Performance:
  - Cache strategically (Redis) for hot reads; background tasks (FastAPI BackgroundTasks/Celery/RQ) for long jobs.

# Frontend (React 19 + TypeScript) rules
- Structure:
  /web
    /app (routes, layouts, error boundaries)
    /features/<name>/ (components, hooks, api.ts, types.ts)
    /components (shared UI)
    /lib (utils, http client, zod schemas)
- Use React 19 features appropriately: Suspense, concurrent rendering; introduce Server Components/Actions only when infra supports them.
- Routing: React Router 6+; ErrorBoundary + Suspense for async UI.
- Data fetching: TanStack Query; colocate feature API code in `api.ts`; avoid fetching inside UI components directly.
- Forms: React Hook Form + Zod (schema-first validation).
- State: prefer server cache (Query) over global state; use Zustand/Context sparingly.
- Types:
  - `strict: true`; avoid `any`.
  - Generate API types from OpenAPI (`openapi-typescript`) and consume them in the client.
- UX/A11y: semantic HTML, ARIA where needed, keyboard navigable; clear loading/skeletons and error displays.
- Styling: consistent system (e.g., Tailwind or a design system). Avoid complex inline styles.

# API design
- RESTful, resource-oriented; plural nouns; predictable filter/sort/pagination.
- Version via `/api/v1`; keep changes backward compatible.
- Time in UTC; ISO-8601 in APIs.

# Error handling pattern (backend & frontend)
- Guard clauses first; early returns for invalid states.
- Keep the "happy path" last; minimize nesting.
- Frontend shows friendly messages; preserves technical details in logs/diagnostics.
- MSW mirrors FastAPI error envelope in tests.

# Testing strategy
- Backend: pytest; coverage ≥ 85% on changed lines.
  - httpx AsyncClient + lifespan; Testcontainers Postgres for integration.
  - Mock externals with `respx`; deterministic fixtures/seeds.
- Frontend: Vitest + React Testing Library; MSW for network; Playwright for critical flows.
  - Test behavior, not implementation details; avoid huge snapshots.

# CI/CD & quality gates
- Lint/format: ruff + black + isort (backend); eslint + prettier (frontend).
- Types: mypy (backend), `tsc --noEmit` (frontend).
- Run unit + integration tests in CI; fail if:
  - DB schema drift (migrations not updated).
  - Public API changed without OpenAPI refresh.
  - New DB models lack constraints/indexes where needed.
- Build artifacts are reproducible; pin tool versions; use pre-commit hooks.

# Conventions & defaults
- Files/dirs: lowercase_with_underscores for Python; kebab-case or PascalCase per frontend norms.
- DB URL: `postgresql+asyncpg://USER:PASS@HOST:5432/DBNAME`
- Pagination query: `?limit=50&offset=0` (max 200) or keyset for large lists.
- Secrets in env, not in code. Provide `.env.example`.
- Logging is structured JSON everywhere.

# Don’ts
- Don’t call the ORM from routers; go through services/use-cases.
- Don’t couple UI components to data fetching.
- Don’t introduce global mutable singletons without a clear need.
- Don’t skip input/output validation or return raw ORM objects.

# Acceptance checklist (every PR)
- [ ] Tests added/updated; CI green; coverage threshold met.
- [ ] OpenAPI/types regenerated and committed.
- [ ] Migrations added/verified; rollback path exists.
- [ ] Logs/errors are actionable; no secrets leaked.
- [ ] Docs updated (README/architecture notes) for significant changes.


When generating or modifying code:
- Do not rename, move, or delete existing files unless I explicitly say so.
- Limit edits to files I mention or to the immediate feature folder. If unsure, ask.
- Before any change that touches >3 files or crosses layers (API, DB, UI), STOP and propose a short plan first:
  1) goal, 2) affected files, 3) data model/migration impact, 4) rollout + fallback.
- Maintain backward compatibility of public APIs, DB schemas, and route params.
  - If a breaking change is necessary, propose a deprecation/migration plan and wait for approval.
- Database: never apply schema changes inline. Generate Alembic migrations and show the migration diff separately.
- Frontend: don’t change route structure, global state shape, or shared component APIs unless asked.
- Tests: add/adjust tests for changed behavior; do NOT rewrite tests to mask regressions.
- Always show a git-style diff for review, with a concise rationale per hunk.
- Preserve existing code style and patterns; prefer minimal diffs that solve the task.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andresfranco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
