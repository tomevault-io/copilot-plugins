## sweepstake

> - **Backend** (`backend/`): FastAPI + SQLModel + Alembic + pytest

# CLAUDE.md

## Stack
- **Backend** (`backend/`): FastAPI + SQLModel + Alembic + pytest
- **Frontend** (`frontend/`): React + TypeScript + Vite + Redux Toolkit + RTK Query + Tailwind CSS
- **Deployment**: Two services — `db` (PostgreSQL) and `app` (nginx + gunicorn + React, managed by supervisord)

## Core Rules
- Make narrow, focused changes. Do not refactor adjacent code, add docstrings, or introduce unused abstractions.
- Reuse existing patterns, naming, and file layout. No new dependencies without clear justification.
- After changes, run the narrowest relevant check (see Validation).

## Backend
- Domain modules: `models.py`, `crud.py`, `routers.py` per feature area.
- **API docs live in `routers.py` only.** Keep endpoint docstrings and field `description=` params up to date when changing behavior. Do not duplicate these descriptions in `crud.py`, `models.py`, or elsewhere.
- Use async DB access. Preserve `Depends(get_db)` and existing auth dependencies.
- **Auth: JWT via HttpOnly cookies** (no Authorization headers). Endpoints use `Depends(verify_access_token)` → extract `token_payload["uid"]`. Never redesign auth flow.
- **Models:** Keep `table=True` SQLModel classes separate from Pydantic request/response schemas (`XCreate`, `XRead`).
- **Model changes — cascade review:** Whenever a `models.py` file is changed (fields added/removed/renamed, types changed, relationships altered), always review the corresponding `crud.py` and `routers.py` in that same domain module and update them as needed. Also update the frontend TypeScript types and RTK Query endpoints. Do not leave any layer inconsistent with the model.
- **CRUD:** Use `selectinload` for relationships (avoid N+1). Call `db.flush()` before accessing auto-generated IDs. Encapsulate reusable filters in a `_query_x()` helper.
- **Never hand-write Alembic migrations.** Change SQLModel definitions instead — the startup workflow autogenerates and applies migrations via the volume at `/app/data/db_migrations`.
- All runtime config lives in `backend/src/config.py` as `SERVER_*` env vars. Add new behavior there, not as hardcoded values.

## Frontend
- Functional components, typed Redux hooks (`useAppSelector`/`useAppDispatch`), RTK Query for all server data — never use raw `fetch()`.
- RTK Query base URL comes from `VITE_API_BASE_URL` env var (`frontend/src/api/baseApi.ts`). 401s auto-refresh via that file — do not duplicate retry logic.
- Reuse existing tag types for cache invalidation; do not invent new tag strings.
- **Type sync with backend models:** Whenever backend models or API schemas change, update the corresponding TypeScript types and RTK Query endpoint definitions in `frontend/src/` to match. Keep request/response types in sync; do not leave stale field names or missing fields in the frontend types.
- Tailwind-only styling. Respect OS light/dark mode — no custom theme toggle unless explicitly requested.
- Fully responsive: iPhone 12 portrait → 4K landscape. Cap content width on wide screens.

## Deployment
- nginx serves the SPA, proxies `/api/` → `127.0.0.1:8888` (gunicorn). FastAPI requires `SERVER_ROOT_PATH=/api`.
- Multi-stage Docker build: Node stage builds frontend assets; Python deps install into `/venv`; runtime image stays slim.
- All environment differences (dev / compose / prod) belong in env vars, not in application code.

## Testing
- Fixtures in `backend/tests/conftest.py`: use `client_user_1` (auth bypassed via dependency override) for fast unit tests; use `client_unauth` only when testing the actual cookie/JWT auth flow.
- Do not hand-write auth tokens or override dependencies inside individual test files — use the existing fixtures.
- **Test coverage for model changes:** After any change to `models.py`, verify existing tests still pass and add or update tests to cover the changed behaviour. At minimum: assert new fields are returned in API responses, accepted in create/update requests, and validated correctly. Run `pytest backend/tests/<touched_area>/` before declaring done.

## Validation
- Backend: `pytest backend/tests/<touched_area>/`
- Frontend: `npm run lint && npm run build` from `frontend/`
- Deployment config: `docker compose config`; full check: `docker compose up --build`
- Do not claim a command was run if it was not.

---
> Source: [vanalmsick/sweepstake](https://github.com/vanalmsick/sweepstake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
