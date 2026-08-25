## clouddrive

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

### Backend (`cd backend`)

```bash
uv sync --all-extras --dev          # install deps
uv run uvicorn app.main:app --reload  # dev server (localhost:8000)

# Quality checks
uv run ruff format --check app tests
uv run ruff check app tests
uv run mypy app tests
uv run pytest                        # all unit tests
uv run pytest tests/integration      # integration tests (needs Postgres)
uv run pytest tests/drive/test_router.py::test_list_items_returns_page  # single test
uv run pytest -k "test_quota"        # filter by name
uv run pytest --cov=app --cov-report=term-missing
```

Integration tests require a running PostgreSQL instance. The default URL is `postgresql+asyncpg://postgres:postgres@localhost:5432/clouddrive_test` (set via `DATABASE_URL` env var or start with `docker compose up postgres`).

### Frontend (`cd frontend`)

```bash
npm ci                    # install deps
npm run dev               # dev server (localhost:5173)
npm run lint
npm run typecheck
npm run test -- --run     # all unit tests (one-shot)
npm run test              # watch mode
npx vitest run src/api/authApi.test.ts   # single file
npx vitest run -t "stores access token"  # filter by test name
npm run build
npm run playwright:install  # first-time only
npm run test:e2e
```

### Docker

```bash
docker compose up --build   # start all services
docker compose up postgres  # Postgres only (for integration tests)
```

## Architecture

### Backend

Every domain is a self-contained package under `backend/app/<module>/` with four files: `router.py`, `service.py`, `repository.py`, `schemas.py`. Modules never import from each other's internals — they import each other's services via FastAPI dependency injection.

```
app/
  core/          config, security (JWT), exceptions, dependencies
  models/        SQLAlchemy ORM models (all imported into Base metadata)
  schemas/       shared Pydantic response types (DriveItemResponse, Page, etc.)
  api/v1/router.py   aggregates all module routers
  <module>/
    router.py    FastAPI routes; depends on service via _module_service factory
    service.py   business logic; takes AsyncSession + other services
    repository.py  DB queries (raw SQLAlchemy)
    schemas.py   module-specific Pydantic I/O schemas
```

**Auth flow:** `get_current_user_id` (in `core/dependencies.py`) extracts a UUID from the JWT Bearer token. Every protected router depends on `CurrentUserId = Annotated[UUID, Depends(get_current_user_id)]`.

**Error handling:** Raise typed subclasses of `AppError` — `NotFoundError` (404), `ForbiddenError` (403), `QuotaExceededError` (413), `NameConflictError` (409), `InvalidOperationError` (422). The global handler in `main.py` converts them to `{ "error": { "code": "...", "message": "...", "details": {} } }`.

**Migrations:** `backend/alembic/` — run `uv run alembic upgrade head` to apply.

**Storage:** `StorageProvider` protocol (in `app/storage/`) abstracts file I/O. `LocalStorageProvider` is the only implementation. The `LOCAL_STORAGE_PATH` env var must be set before app import because `get_settings()` is `@lru_cache`.

**Data model notes:**
- `user_item_preferences` table stores per-user starred state (not a column on `drive_items`)
- "Recent" items are derived from `activity_logs`, not `drive_items.updated_at`
- Refresh token and public share token are stored as hashes only

### Frontend

```
src/
  api/           authApi, driveApi, uploadApi, etc. — thin wrappers over axios
  app/           AuthInitializer, RequireAuth, RedirectIfAuth, router, ProtectedLayout
  stores/        authStore (Zustand) — access token only; uiStore, uploadStore
  hooks/         useAuth, useDrive, useUpload, etc. — TanStack Query wrappers
  pages/         one file per route
  components/    drive/, layout/, preview/, share/, trash/, upload/
```

**Auth / token lifecycle:**
- `authStore` holds the access token in memory only — no localStorage, no sessionStorage, no Zustand persist middleware.
- On every page load, `AuthInitializer` (`src/app/AuthInitializer.tsx`) calls `POST /auth/refresh` before the router renders. If the HttpOnly refresh token cookie is still valid, the access token is restored in Zustand and the user stays on their page. Until the attempt settles, `AuthInitializer` returns `null` to prevent `RequireAuth` from prematurely redirecting.
- `RequireAuth` then just reads `authStore.accessToken` — non-null means authenticated.
- The Axios interceptor in `client.ts` handles 401s on subsequent API calls by calling refresh again and retrying. It uses a separate `refreshClient` (no interceptors) to avoid infinite loops.

**API client:** `src/api/client.ts` exports `api` (main Axios instance) and `refreshClient` (interceptor-free, for refresh calls). `toApiError()` reads `d['code']` directly from the response body — MSW error handlers must return `{ code: '...', message: '...' }` flat, not nested.

**State:** Server state via TanStack Query (v5); UI/upload/auth state via Zustand (v5). Query keys are defined per-hook file using factory functions (`driveKeys.items(parentId)` etc.).

## Testing Patterns

> **User-visible changes must be verified in a real browser before they count as done.**
> Unit tests run against jsdom and MSW mocks; the app runs against a real backend
> in a real browser. Open the page with the browser tools, drive the actual flow,
> and check the console and the requests that were really sent — confirming an
> element exists is not verification. Report what you did and what you saw, and
> record it in the module's `doc/tasks/<module>.md`. Never ask the user to check
> manually, and never use throwaway-free real data for destructive steps: create a
> disposable account and folder.
>
> This exists because it has already gone wrong twice here: a PDF preview declared
> fixed after only checking that an `<iframe>` rendered (it still could not
> preview), and proposal §33's backend shipped with no frontend at all (an editor
> link opened read-only). Both had green unit suites.

### Backend unit tests

Each module's `tests/<module>/test_router.py` follows this pattern — no real DB needed:

```python
def _make_app(service: SomeService, user_id: UUID) -> FastAPI:
    app = FastAPI()
    app.add_exception_handler(AppError, ...)      # inline error handler
    app.dependency_overrides[get_db] = _fake_db   # AsyncMock session
    app.dependency_overrides[_some_service] = lambda: service
    app.include_router(some_router)
    return app

async def test_something(user_id, headers):
    svc = AsyncMock(spec=SomeService)
    svc.some_method.return_value = expected
    app = _make_app(svc, user_id)
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as c:
        resp = await c.get("/endpoint", headers=headers)
    assert resp.status_code == 200
```

Auth header: `{"Authorization": f"Bearer {create_access_token(user_id)}"}` — creates a real JWT, no mocking needed.

### Backend integration tests

Live in `tests/integration/`. Require Postgres. Each test gets a clean DB via `TRUNCATE ... RESTART IDENTITY CASCADE` (automatic via the `_truncate_tables` fixture in `tests/integration/conftest.py`). Use `register_and_login(client)` helper to get a real access token. `LOCAL_STORAGE_PATH` must be set before any app import — the conftest handles this.

### Frontend unit tests

Use Vitest + MSW. Per-test MSW overrides: `server.use(http.get(...))` inside the test. Shared baseline handlers live in `src/test/handlers.ts`.

- Component tests **must** call `afterEach(() => cleanup())` explicitly — Vitest does not auto-clean the DOM.
- `queryByRole('cell')` also matches `<th>` — use `document.querySelectorAll('tbody td')` to check for empty table bodies.
- Axios encodes spaces as `+` in query strings — use `new URL(url).searchParams.get('q')` instead of `decodeURIComponent` in MSW handlers.
- `request.formData()` fails in MSW handlers with jsdom `File` objects (undici incompatibility) — use `request.text()` to inspect raw multipart bodies.

## Design Documents

`doc/` contains the authoritative design for the project:

| File | Purpose |
|---|---|
| `doc/prompt.md` | Codex multi-agent orchestration prompt + confirmed technical decisions |
| `doc/detailed-design.md` | Module-level architecture, service interfaces, API contracts |
| `doc/proposal.md` | Product requirements and feature design |
| `doc/tasks/<module>.md` | Per-module task checklist (checked = implemented + tested) |
| `doc/tasks/progress.md` | Overall 28-module completion status |
| `doc/decisions.md` | Architecture decision records (DEC-XXX format) |

**After adding any new feature**, update `doc/prompt.md` (Stage extra requirements + file ownership table), `doc/detailed-design.md` (relevant module section), and the affected `doc/tasks/<module>.md` (new items, checked).

---
> Source: [billwu101/CloudDrive](https://github.com/billwu101/CloudDrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
