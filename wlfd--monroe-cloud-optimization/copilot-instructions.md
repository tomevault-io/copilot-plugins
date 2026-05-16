## monroe-cloud-optimization

> CloudCost is an Azure cloud cost optimization SaaS platform. It ingests Azure billing data, detects spending anomalies, generates AI-powered recommendations (via Anthropic Claude or Azure OpenAI), and provides per-tenant cost attribution with budgets and alerts. FastAPI backend (Python 3.12) + React 19/TypeScript frontend, backed by PostgreSQL 15 and Redis 7.

# CLAUDE.md

## Project Overview

CloudCost is an Azure cloud cost optimization SaaS platform. It ingests Azure billing data, detects spending anomalies, generates AI-powered recommendations (via Anthropic Claude or Azure OpenAI), and provides per-tenant cost attribution with budgets and alerts. FastAPI backend (Python 3.12) + React 19/TypeScript frontend, backed by PostgreSQL 15 and Redis 7.

## Quick Commands

```bash
# Dev environment — starts DB, Redis, runs migrations, seeds admin user, starts API + frontend
docker compose up --build          # Start all services
docker compose down                # Stop all services
docker compose down -v             # Stop and delete database volume
# Default login: admin@cloudcost.local / admin123 (auto-seeded on first run)

# Backend (manual)
cd backend && pip install -r requirements.txt -r requirements-dev.txt
alembic upgrade head               # Run migrations
uvicorn app.main:app --reload      # Start API server (port 8000)

# Frontend (manual)
cd frontend && npm ci
npm run dev                        # Start dev server (port 3000)

# Testing (Docker-based via Makefile)
make test                          # Run all tests
make test-backend                  # Backend only (pytest via Docker)
make test-frontend                 # Frontend only (vitest via Docker)
make test-backend-coverage         # Backend with coverage
make test-frontend-coverage        # Frontend with coverage

# Linting & formatting (Docker-based via Makefile)
make lint                          # Check all (ruff + eslint)
make lint-fix                      # Fix all
make format                        # Format all (ruff format + prettier)

# Seed admin user
docker compose exec api python -m app.scripts.seed_admin
```

## Architecture

- **Backend**: `backend/app/` -- FastAPI, layered as `api/v1/` (routes) -> `schemas/` (Pydantic) -> `services/` (business logic) -> `models/` (SQLAlchemy ORM async)
- **Frontend**: `frontend/src/` -- React 19, `pages/` (route components) -> `services/` (API hooks via TanStack Query + Axios) -> `hooks/` (React context) -> `components/` (shadcn/ui primitives)
- **Database**: PostgreSQL 15 via async SQLAlchemy 2.0 + Alembic migrations (auto-run by `migrate` Docker service)
- **Cache**: Redis 7 for recommendation caching and job deduplication
- **Scheduler**: APScheduler in-process -- ingestion (4h interval), recommendations (daily 02:00 UTC), budget checks (4h, offset 1h after ingestion), webhook retry (15min)
- **Auth**: JWT access tokens (60min, in-memory only) + refresh tokens (7 days, HttpOnly cookie). Four roles: `admin`, `devops`, `finance`, `viewer`

## Key Patterns to Follow

- **Backend routes**: Add new routes in `backend/app/api/v1/`, register in `router.py`. Use `Depends(get_db)` and `Depends(get_current_user)`.
- **Backend services**: Business logic in `backend/app/services/`. Accept `AsyncSession` as first param. Use `session.execute()` with SQLAlchemy select statements.
- **Pydantic schemas**: Request/response models in `backend/app/schemas/`. All API responses go through schemas.
- **Frontend pages**: New pages in `frontend/src/pages/`, add route in `App.tsx` (`createBrowserRouter`).
- **Frontend API hooks**: TanStack Query hooks in `frontend/src/services/`. Pattern: `useQuery` for reads, `useMutation` for writes, invalidate queries on success. Axios instance in `services/api.ts`.
- **Database migrations**: `cd backend && alembic revision --autogenerate -m "description"` then `alembic upgrade head`.
- **Environment**: Copy `.env.example` to `.env.local` (Docker) or `backend/.env` (manual). Set `MOCK_AZURE=true` for local dev without Azure credentials.

## Testing Patterns

- **Backend tests**: `backend/tests/` -- pytest-asyncio, mock DB with `AsyncMock`. Helper fixtures `make_scalars_result` / `make_scalar_result` in `conftest.py` for mocking SQLAlchemy query results.
- **Frontend tests**: `frontend/src/test/` -- Vitest + React Testing Library + MSW. Custom `render()` from `test-utils.tsx` wraps providers. Mock handlers in `mocks/handlers.ts`.
- **Run single test**: `cd backend && pytest tests/test_cost_service.py -v` or `cd frontend && npx vitest run src/test/pages/DashboardPage.test.tsx`

## Important Files

- `backend/app/main.py` -- App factory, lifespan, middleware, scheduler registration
- `backend/app/core/config.py` -- All settings (Pydantic Settings from env vars)
- `backend/app/core/dependencies.py` -- FastAPI dependency injection (`get_db`, `get_current_user`)
- `backend/app/models/__init__.py` -- All model imports (Alembic autogenerate reads from here)
- `frontend/src/App.tsx` -- React Router setup, route definitions
- `frontend/src/services/api.ts` -- Axios instance, auth interceptors, token refresh
- `docker-compose.yml` -- Dev environment (PostgreSQL, Redis, migrate, API, frontend)
- `docker-compose.tools.yml` -- Used by Makefile for lint/test/format commands
- `.env.example` -- All environment variables documented

## Common Gotchas

- Backend uses `async` everywhere -- don't use sync SQLAlchemy patterns
- Frontend access tokens are in-memory only (not localStorage) -- cleared on page refresh, refresh token cookie handles re-auth
- `docker compose up` auto-seeds an admin account (`admin@cloudcost.local` / `admin123`) and enables `MOCK_AZURE` — no external credentials needed
- `MOCK_AZURE=true` returns synthetic data; set to `false` + provide `AZURE_*` credentials for real data
- Alembic `env.py` imports all models from `app/models/__init__.py` -- new models must be added there or autogenerate won't detect them
- The ingestion lock (`_ingestion_lock` in `services/ingestion.py`) is `asyncio.Lock()`, process-local -- doesn't work across multiple instances
- The Makefile uses `docker-compose.tools.yml` (not the main `docker-compose.yml`) for lint/test/format targets

---
> Source: [wlfd/monroe_cloud_optimization](https://github.com/wlfd/monroe_cloud_optimization) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
