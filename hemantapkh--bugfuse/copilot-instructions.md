## bugfuse

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

BugFuse is a lightweight, self-hosted error tracking platform (a Sentry-compatible
alternative). It ingests events from Sentry SDKs via the Sentry envelope/store
protocols, groups them into issues, and surfaces them through a web UI with alerting
automations. It is a monorepo: `backend/` (FastAPI + async SQLAlchemy + Postgres),
`frontend/` (React 19 + Vite + Tailwind v4), and `website/` (public site: Astro +
Starlight docs + Tailwind v4).

## Commands

### Backend (run from `backend/`, uses `uv`)
- Install deps: `uv sync`
- Run dev server (autoreload, 127.0.0.1:8000): `uv run dev` (defined as a project script; `uv run start` runs without reload)
- Lint: `uv run ruff check`. **Do NOT run `uv run ruff format`** — the pinned ruff (0.15.6) has a formatter bug that strips the parentheses from multi-type `except (A, B):` clauses, producing invalid Python 3 (this is the original source of the `except A, B:` errors that were fixed). Use `ruff check --fix` for import sorting; format by hand until ruff is repinned.
- Tests: `uv run pytest` (from `backend/`). The suite builds a real Postgres schema from Alembic migrations; by default it uses **Testcontainers** (needs Docker), or set `TEST_DATABASE_URL=postgresql+asyncpg://…` to point at an existing Postgres. Each test runs in a rolled-back transaction. See `backend/tests/README.md`. The app targets Python **3.14** (it uses `uuid.uuid7` and PEP 649 deferred annotations); the suite also runs on 3.13 via small shims in `tests/conftest.py`.
- Type check: the installed dev dependency is **mypy** (`uv run mypy app`), but the pre-commit *pre-push* hook invokes Astral's `ty check app tests`. `ty` is **not** in `uv.lock`/deps — that hook relies on it being available externally. Confirm which checker is intended before relying on either.
- DB migrations: `uv run alembic upgrade head` — Create a migration: `uv run alembic revision --autogenerate -m "message"`
- API docs served at `/api/docs` (Swagger) and `/api/redoc` once running.

### Frontend (run from `frontend/`, uses `pnpm`)
- Install: `pnpm install`
- Dev server (port 3000, proxies `/api` → `localhost:8000`): `pnpm dev`
- Build (runs typecheck first): `pnpm build`
- Combined gate: `pnpm check` (= `pnpm typecheck && pnpm lint`)
- Format: `pnpm format` (Prettier)

### Website (run from `website/`, uses `pnpm`)
- Public site (landing + docs) at bugfuse.com: Astro with Starlight docs at `/docs`.
- Install: `pnpm install` — Dev server: `pnpm dev` — Build: `pnpm build`
- Landing page: `src/pages/index.astro`; docs are markdown in `src/content/docs/docs/`.

### Full stack
- `docker compose up` brings up Postgres, runs migrations (`migrate` service), then backend and frontend. App is exposed on `BUGFUSE_PORT` (default 3000).

### Conventions enforced by pre-commit / CI
- Commits **must** be Conventional Commits, `--strict`, with scope from `{backend, frontend, website, infra, ci}` and type from `{build, chore, ci, docs, feat, fix, refactor, test}`. Example: `feat(backend): add release tracking`.
- `ruff check`/`ruff format` run on commit; `ty check` runs on push. Install hooks with `pre-commit install` (it installs `pre-commit`, `pre-push`, and `commit-msg` stages).
- Python target is **3.14**; ruff line-length 100 (E501 ignored), double quotes.

## Backend architecture

### Module layout convention
Each domain under `backend/app/<domain>/` follows a consistent split:
`models.py` (SQLAlchemy ORM), `schemas.py` (Pydantic I/O), `router.py` (FastAPI
endpoints), `service.py` (business logic), `queries.py` (read queries),
`dependencies.py` (FastAPI `Depends` guards). Routers are wired together in
`app/main.py`. Stick to this layering when adding features — keep DB/business
logic out of routers.

**Target layering.** Every HTTP/CRUD domain has been migrated onto a single
convention so business logic is unit-testable without HTTP (`projects`,
`organizations`, `invites`, `auth`, `auth/tokens`, `notification_channels`,
`views`, `events`, `issues` + `activities`, `environments`, `automations`
rules/executions, `sentry/artifacts`, `sentry/ingest` DSN auth). The
`sentry/ingest` request-scoped DSN credential check (`get_sentry_project`) is
migrated — it uses `TransactionalSession`, a `queries.py` lookup, and domain
errors — but the actual ingest hand-off after auth is still fire-and-forget.
The remaining fire-and-forget paths (`event_processing`, the automation
engine/dispatcher, post-ingest symbolication, and the bearer-token auth lookup)
intentionally keep managing their own sessions/transactions and still use the
legacy `DatabaseSession`. The convention:
- `router.py` — HTTP only: dependency wiring, request/response schemas, status
  codes. No SQLAlchemy queries, no `db.begin()`, no `HTTPException`.
- `service.py` — business logic: `(db, *, typed args)` in, ORM/DTO out, raises
  **domain errors** (`app/shared/errors.py`: `DomainError` →
  `ValidationError`/`NotFoundError`/`ConflictError`/`PermissionDeniedError`/
  `GoneError`). Never raises `HTTPException`. Does not own session lifecycle.
- `queries.py` — non-trivial read queries (session in, data out).
- `dependencies.py` — thin authz/resource loaders sharing the request session.
- **Transaction boundary** is request-scoped: handlers/deps depend on
  `TransactionalSession` (`app/core/database`), which opens one transaction per
  request (commit on success, rollback on error). Migrated code must NOT call
  `db.begin()`. The legacy `DatabaseSession` (manual `db.begin()`) still exists
  for un-migrated modules; both coexist during the migration.
- **Error mapping** is centralized: a single `DomainError` handler in
  `app/main.py` renders `{"detail": ...}` with the right status code, so routers
  don't `try/except`. `app/automations/exceptions.py` folds into this hierarchy.

### Auth & multi-tenancy
- Cookie-based JWT (`access_token` cookie, also accepted as Bearer). Decode logic in `app/core/security.py`; current-user dependency in `app/auth/dependencies.py` (`CurrentUser`).
- Data is scoped **Organization → Project → Issue/Event**. Authorization is enforced through dependency guards: `app/organizations/dependencies.py` provides `CurrentOrgMembership` and `require_org_role(...)` (roles in `app/shared/enums.py`: OWNER/ADMIN/MEMBER). Most routes are keyed by `org_id`/`project_id` path params.
- `app/auth/tokens/` = long-lived API auth tokens (for the Sentry upload/CLI API); `app/auth/action_tokens.py` = signed single-use tokens for email flows (signup verify, password reset, invites).

### Event ingestion pipeline (the core of the system)
This is the most important subsystem; understanding it requires reading several files together.
- **Entry points** (`app/sentry/ingest/router.py`): `POST /api/{project_id}/envelope/` and `/store/`. These authenticate the project via the Sentry DSN key (`app/sentry/ingest/auth.py` parses `X-Sentry-Auth` / `sentry_key`), then hand the raw bytes to a FastAPI `BackgroundTask` and return `204` immediately. **Ingestion is fire-and-forget**; nothing is awaited in the request.
- **Parsing** (`app/event_processing/parsing/`): unwraps the Sentry envelope/store payloads into event dicts.
- **Pipeline** (`app/event_processing/`): `service.ingest_event` runs `EventPipeline` (`pipeline.py`) — an ordered tuple of async **stages** (`stages/__init__.py:DEFAULT_STAGES`). Each stage mutates a shared `PipelineContext` (`context.py`) and may set `ctx.skip_remaining` to short-circuit. Stage order matters: normalize → dedup → timestamp → release → environment → **fingerprint/grouping** → resolve issue → persist event → update issue summary → state transitions → activity → affected users.
- **Grouping**: `app/event_processing/fingerprinting/` computes the fingerprint that maps an event to an existing or new issue (this is what "groups" errors).
- **Post-ingest side effects** are dispatched as detached `asyncio.create_task`s after the DB transaction commits (in `service._ingest_payloads`): automation evaluation and (if applicable) symbolication. Don't add awaited heavy work into the pipeline; follow this fire-and-forget pattern.

### Automations / alerting (`app/automations/`)
Event → rule evaluation → action dispatch. `RuleEngine` (`engine.py`) matches an
`EventContext` against `AutomationRule`s (filters + conditions like threshold/frequency/spike).
`AutomationDispatcher` (`dispatcher.py`) executes matched actions (currently `notify`)
via pluggable **executors** (`executors/`), records one `AutomationExecution` per rule,
and is rate-limited (`rate_limiter.py`). The singleton `automation_service` (`__init__.py`)
has `startup()`/`shutdown()` hooks called from the app lifespan. Notification delivery
goes through `app/notification_channels/` providers (email, slack, discord, telegram,
webhook, http, apprise).

### Issue search (`app/issues/search/`)
A Sentry-style query language (`is:unresolved level:error foo`). `parser.py` tokenizes
the query; `filters/` + `registry.py` map tokens to SQL conditions; `apply.py` applies
them to the issues query. Add new searchable fields via the filter registry, not ad-hoc
query code.

### Debug artifacts & symbolication (`app/debug_artifacts/`, `app/sentry/artifacts/`)
Source maps / debug files are uploaded via the Sentry chunk-upload API
(`app/sentry/artifacts/`) and stored through a pluggable `ArtifactStorage` abstraction
(`debug_artifacts/storage.py`) backed by **local filesystem or S3**, chosen by config
(`ARTIFACT_STORAGE_S3_BUCKET`). Symbolication (resolving minified/native stack traces)
runs as a post-ingest background task using the `symbolic` library.

### Database
- Async SQLAlchemy 2.0 + asyncpg. Engine/session in `app/core/database/session.py`; `DatabaseSession` is the FastAPI-injected session. Note services often manage their own `async with db.begin()` transaction blocks.
- All models must be imported in `app/core/database/models.py` so they register on `Base.metadata` (alembic autogenerate and table creation depend on this). Migrations live in `backend/alembic/versions/`; `alembic/env.py` supports both Postgres and SQLite.
- `DATABASE_URL` may be a sync-style URL; `make_async_db_url` (`app/core/database/utils.py`) adapts the driver.

### Config
All settings come from env via `app/core/config.py` (`pydantic-settings`, reads `.env`).
The `settings` singleton is imported widely. Key groups: JWT/security, SMTP email,
cache (`redis`/`aiocache`), Sentry ingest size limits & proxy-header trust, artifact
storage. Email sending is a no-op unless `SMTP_HOST` + `SMTP_FROM` are set (`settings.smtp_enabled`).

## Frontend architecture

- **Stack**: React 19, React Router v7, TanStack Query (server state), Axios, Tailwind v4, react-icons. No component library — UI primitives live in `src/components/ui/`. Path alias `@/` → `src/`.
- **Structure**: feature-first under `src/features/<feature>/` (auth, issues, projects, channels, automations, settings), each with `components/`, `hooks/`, and local `form.ts`/`constants.ts`/`types.ts`. Shared API calls in `src/api/` (one file per resource, all using the shared `client` in `src/api/client.ts`, which is configured with `withCredentials` for cookie auth). Routes are defined in `src/app/router.tsx` with lazy-loaded pages and `ProtectedRoute`/`GuestRoute` wrappers.
- **Auth/theme** are React contexts (`src/contexts/`). Server state is fetched/cached via TanStack Query hooks (`useXxx` in feature `hooks/` dirs); prefer adding a query/mutation hook over calling the API client directly in components.
- **Platform onboarding data**: `src/data/platforms/` contains per-SDK setup/DSN instructions surfaced in the project setup flow.

## Notes
- Tests live under `backend/tests/` mirroring `app/`. The suite uses `pytest` + `pytest-asyncio` against a real Postgres (Alembic-migrated schema, per-test transaction rollback) with an `httpx` ASGI client; fixtures (`tests/conftest.py`) provide `db_session`, `client`/`authenticated_client`, and `user`/`org`/`project`/`issue`/`invite` factories. Two test tiers per migrated module: HTTP characterization tests (`test_<domain>_api.py`) and service/query unit tests (`test_<domain>_service.py`). Run with `uv run pytest`. See `backend/tests/README.md`.

---
> Source: [hemantapkh/bugfuse](https://github.com/hemantapkh/bugfuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
