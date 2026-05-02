## fitness-api

> Instructions for AI coding agents working on this codebase. Read this before making changes.

# AGENTS.md

Instructions for AI coding agents working on this codebase. Read this before making changes.

## Verification

Always run these before considering work done:

```bash
make lint          # Ruff linter (must pass)
make ty            # Type checker (must pass)
make test          # Unit tests (must pass, requires JWT_ISSUER env var)
make e2e-test      # E2E tests (requires Docker)
```

Unit tests need `JWT_ISSUER` set (any value works, e.g. `JWT_ISSUER=http://localhost:8080 make test`). CI sets this automatically.

## Architecture: How the Layers Fit Together

Requests flow through four layers: **Router -> DB -> Model**, with routers in `fitness/app/routers/`, database functions in `fitness/db/`, and Pydantic models in `fitness/models/`.

There is no ORM. All database access is raw SQL via psycopg3.

### Adding a New Endpoint

1. **Model** (`fitness/models/thing.py`): Define Pydantic BaseModel classes for the domain object.

2. **DB functions** (`fitness/db/thing.py`): Write query functions using `get_db_cursor()`:
   ```python
   from .connection import get_db_cursor

   def get_things() -> list[Thing]:
       with get_db_cursor() as cursor:
           cursor.execute("SELECT id, name FROM things WHERE deleted_at IS NULL")
           return [_row_to_thing(row) for row in cursor.fetchall()]

   def _row_to_thing(row) -> Thing:
       id, name = row
       return Thing(id=id, name=name)
   ```
   - Always use `get_db_cursor()` context manager (auto-commits on success, rolls back on error).
   - Use `get_db_connection()` with `conn.transaction()` for multi-statement transactions (see `fitness/db/shoes.py:merge_shoes` for the pattern).
   - Convert rows to models via `_row_to_*()` helper functions (tuple unpacking, not dict access).
   - Use `psycopg.sql.SQL` for dynamic query construction, `%s` placeholders for parameters.

3. **Router** (`fitness/app/routers/thing.py`):
   ```python
   from fastapi import APIRouter, Depends
   from fitness.app.auth import require_viewer, require_editor

   router = APIRouter(prefix="/things", tags=["things"])

   @router.get("/", response_model=list[Thing])
   def read_things(_user: User = Depends(require_viewer)) -> list[Thing]:
       return get_things()
   ```

4. **Register the router** in two places:
   - `fitness/app/routers/__init__.py`: Add `from .thing import router as thing_router` and add to `__all__`.
   - `fitness/app/app.py`: Add `app.include_router(thing_router)`.

5. **Migration** (if new table needed): `uv run alembic revision -m "add things table"`, then write raw SQL in `upgrade()` and `downgrade()`. See `alembic/versions/` for examples. Migrations use `op.execute()` with raw SQL, not Alembic's `op.create_table()`.

### Auth on Endpoints

- GET endpoints: `_user: User = Depends(require_viewer)` (any authenticated user).
- POST/PATCH/DELETE endpoints: `user: User = Depends(require_editor)` (editor role only).
- `/health` is public (no auth).
- The `require_viewer_or_api_key` dependency supports both OAuth and X-API-Key header (used by the TRMNL summary endpoint).

### Request/Response Models

- Route-specific request/response models go in the router file or in `fitness/app/models.py`.
- Domain models go in `fitness/models/`.
- Use `response_model=` on route decorators.

## Database Conventions

- **Soft deletion**: Use `deleted_at` timestamp, never `DELETE FROM`. Filter with `WHERE deleted_at IS NULL` in queries.
- **Deterministic IDs**: Strava runs use `strava_{id}`, MMF runs use `mmf_{id}`, shoes use a normalized name slug. Not UUIDs (except for users).
- **Timestamps**: Stored in UTC. Timezone conversion happens in the API layer (`fitness/utils/timezone.py`).
- **Version tracking**: The `runs_history` table records every edit (immutable append-only log).

## Testing Conventions

### Unit tests (`tests/app/`, `tests/db/`, etc.)

- **No database access allowed.** An autouse fixture in `tests/conftest.py` patches `psycopg.connect` to raise `AccidentalDatabaseAccessError`. If your test needs the DB, mark it `@pytest.mark.e2e`.
- **Mock at the import location**, not where the function is defined. For example, to mock `get_shoes` used in the shoes router: `monkeypatch.setattr("fitness.app.routers.shoes.get_shoes", ...)`.
- `tests/app/conftest.py` has autouse fixtures that mock common DB calls (runs, shoes) to return empty lists. Override specific mocks in your test with `monkeypatch.setattr`.
- Auth fixtures: `client` (no auth), `viewer_client` (viewer role), `editor_client` (editor role), `auth_client` (editor, legacy alias), `api_key_client` (X-API-Key header).
- Test unauth behavior with `client`, test forbidden behavior with `viewer_client` on editor-only endpoints.

### Test factories (`tests/_factories/`)

- Factories create model instances with sensible defaults: `RunFactory`, `LiftFactory`, `ExerciseFactory`, `SetFactory`, `StravaActivityWithGearFactory`, etc.
- Use `.make(update={...})` to create variations: `run_factory.make({"distance": 10.0})`.
- Session-scoped factory fixtures are available: `run_factory`, `strava_activity_with_gear_factory`, `mmf_activity_factory`.

### E2E tests (`tests/e2e/`)

- Marked with `@pytest.mark.e2e`.
- Use testcontainers to spin up a real Postgres instance, run Alembic migrations, then test against it.
- Session-scoped fixtures: `viewer_client`, `editor_client`, `auth_client` (all with real DB, mocked OAuth).
- Require Docker to run (`make e2e-test`).

### Test file organization

- Router tests go in `tests/app/{router_name}/test_{feature}.py` (e.g., `tests/app/shoe_router/test_delete_endpoint.py`).
- DB tests go in `tests/db/test_{module}.py`.
- Each test file has a module docstring describing what it tests.

## Code Style

- Formatted and linted by **ruff** (`make format`, `make lint`).
- Type-checked by **ty** (`make ty`). Use `# type: ignore[error-code]` sparingly and only with the specific error code.
- Type hints on all function signatures. Use `X | None` (not `Optional[X]`) for new code.
- `from __future__ import annotations` is **not** used in this codebase.
- Imports: stdlib, then third-party, then local `fitness.*` imports. No blank lines between groups within each section.

## Environment

- Config is loaded early via `fitness/app/env_loader.py` (imported first in `app.py`).
- `.env.dev` is the local dev config. `.env.dev.example` is the template.
- Required env vars are validated at startup; the app exits with a clear message if any are missing.
- `JWT_ISSUER` and `IDENTITY_PROVIDER_URL` can differ: `IDENTITY_PROVIDER_URL` is the internal network address (K8s DNS), while `JWT_ISSUER` is the public URL that appears in tokens.

---
> Source: [eswan18/fitness-api](https://github.com/eswan18/fitness-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
