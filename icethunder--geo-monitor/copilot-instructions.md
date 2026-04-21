## geo-monitor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GEO Monitor — 品牌在AI模型（ChatGPT、Claude、Gemini等）中的曝光度、准确性和情感实时监控平台。

## Tech Stack

- **Frontend**: Next.js 14 (App Router) + TypeScript + TailwindCSS + Shadcn/Radix UI
- **Backend**: FastAPI (Python) + SQLAlchemy 2.0 + Pydantic V2
- **Database**: PostgreSQL (Supabase production) / SQLite (local dev fallback)
- **Cache/Queue**: Redis (Upstash production)
- **LLM Routing**: OpenRouter API (multi-model orchestration)
- **Real-time**: WebSocket (native)
- **Auth**: JWT (python-jose + bcrypt)
- **Migration**: Alembic
- **CI/CD**: GitHub Actions

## Development Commands

### Frontend (from `frontend/`)
```bash
npm run dev          # Start dev server on :3000
npm run build        # Production build
npm run lint         # ESLint
npx playwright test  # E2E tests (Chromium, auto-starts dev server)
npx playwright test tests/auth.spec.ts  # Run single test file
```

### Backend (from `backend/`)
```bash
pip install -r requirements.txt              # Install dependencies
python -m uvicorn app.main:app --reload      # Start dev server on :8000
pytest                                        # Run tests
pytest tests/test_auth.py -v                 # Run single test file
```

### Docker (full stack)
```bash
docker-compose up              # Start all services (postgres, redis, backend, frontend, worker, grafana)
docker-compose up backend      # Start backend + dependencies only
```

## Architecture

### Backend (`backend/app/`)
```
main.py              — FastAPI app entry, lifespan management, router registration
core/
  config.py          — Pydantic Settings, all env vars
  security.py        — JWT create/decode, bcrypt password hashing
  dependencies.py    — FastAPI dependency injection (get_current_user, require_role, etc.)
  exceptions.py      — Custom exception hierarchy (AppException → NotFoundException, AuthenticationException, etc.)
api/
  auth_routes.py     — /api/auth/* (register, login, refresh, logout, password reset)
  protected_tasks.py — /api/tasks/* (CRUD + trigger execution, requires auth)
  protected_metrics.py — /api/metrics/* (dashboard data, requires auth)
  protected_config.py — /api/config/* (tenant config CRUD, requires auth, admin for PUT)
  user_management.py — /api/users/* (tenant members, invitations)
models/
  entities.py        — Core SQLAlchemy models (monitor_tasks, task_runs, metrics_snapshot, etc.)
  user_entities.py   — User/Tenant models (users, tenants, user_tenants, sessions)
  database.py        — Engine creation, session factory, init_db/close_db
  schemas.py         — Pydantic request/response DTOs
services/
  auth_service.py    — Registration, token generation, session management
  scheduler.py       — Redis-based task queue, schedule_task/trigger_task_run
  executor.py        — Task execution logic
  calculator.py      — Metrics computation
middleware/
  auth.py            — Role/permission decorators, role hierarchy (owner > admin > member > viewer)
```

### Frontend (`frontend/src/`)
```
app/                 — Next.js App Router pages (auth/, dashboard/, tasks/, analytics/, settings/, users/)
components/
  auth/              — AuthProvider (React Context), AuthGuard (route protection)
  layout/            — MainLayout (sidebar + header), GoogleLayout
  ui/                — Shadcn UI components (button, card, dialog, table, etc.)
  dashboard/         — MetricCard, TrendChart
  charts/            — Recharts/Chart.js visualizations
  tasks/             — TaskList, TaskTable
  notifications/     — NotificationProvider (wraps WebSocket)
lib/
  api/client.ts      — Axios instance with auth interceptor (auto-injects Bearer token, handles 401)
  api/tasks.ts       — Task CRUD service functions
  api/metrics.ts     — Dashboard/analytics service functions
  api/alerts.ts      — Alert service functions
  api/types.ts       — Shared TypeScript interfaces (Task, Alert, PaginatedResponse, etc.)
  auth.ts            — Token helpers (getAccessToken, authenticatedFetch, refreshAccessToken)
  websocket.ts       — WebSocketClient class with reconnection logic
  utils.ts           — cn() for Tailwind class merging, date/number formatting helpers
```

### Key Patterns

- **Multi-tenancy**: Users belong to tenants via `user_tenants` junction table with roles. Tenant context is threaded through JWT claims and dependency injection.
- **Auth flow**: Login → JWT access token (30min) + refresh token (7d) stored in localStorage. Axios interceptor auto-attaches Bearer header. 401 response triggers logout redirect.
- **Dev mode bypass**: When `ENVIRONMENT=development`, authentication can be skipped — returns mock user/tenant.
- **State management**: React Context for auth/notifications, useState + direct Axios calls for data fetching.
- **API prefix**: All backend routes are under `/api`. Frontend Axios client has `baseURL` set to `NEXT_PUBLIC_API_URL`.
- **Path alias**: Frontend uses `@/*` → `./src/*` in imports.

## Environment Setup

Copy `.env.example` files in both `backend/` and `frontend/`. Key variables:

**Backend**: `DATABASE_URL`, `SECRET_KEY`, `OPENROUTER_API_KEY`, `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`, `ENVIRONMENT`

**Frontend**: `NEXT_PUBLIC_API_URL` (defaults to `http://localhost:8000`)

## API Documentation

Backend auto-generates OpenAPI docs at `/docs` (Swagger) and `/redoc`.

## Project Status (2026-02-09)

### MVP Completion: ~95%
All P0/P1/P2 consolidation tasks complete. Core monitoring, auth, dashboard, task management, tests, CI/CD, and Alembic all functional.

### What's Working
- Full JWT auth with multi-tenancy and RBAC (owner/admin/member/viewer)
- Task CRUD with keyword & model associations, cron scheduling
- Metrics calculation engine (SOV, accuracy, sentiment, citation rate, positioning hit)
- Dashboard with real API data + Recharts visualizations (SOV trend, accuracy trend, model comparison, keyword performance)
- Alert system (threshold-based webhook + email + in-app NotificationCenter, configurable SOV threshold)
- Google-style responsive layout with global search, breadcrumbs, mobile sidebar, dark mode
- User management with invitations and role assignment
- Task execution framework (APScheduler + Redis queue + OpenRouter integration)
- Docker Compose full-stack orchestration (postgres, redis, backend, frontend, worker, grafana)
- Email service (verification, password reset, alert notifications via aiosmtplib)
- Forgot password page (frontend + backend)
- Protected config API (tenant-scoped, admin-only write, base64 API key encryption)
- Backend test suite: pytest for auth (13 tests), tasks (15 tests), calculator (34 tests), alerts (22 tests)
- Alembic database migrations initialized
- GitHub Actions CI (backend lint+test, frontend lint+build)
- WebSocket real-time push wired through task execution → NotificationCenter
- LLM executor with base64 API key decryption, markdown fence stripping, robust JSON parsing

### Known Gaps
- LLM executor not validated end-to-end against real OpenRouter API (logic improved but needs live test)
- Backend tests not yet run in CI (need `pip install` + `pytest` verification)
- No Alembic initial migration generated yet (`alembic revision --autogenerate`)
- WebSocket JWT validation still basic (uses decode_token fallback)
- Frontend E2E tests are skeletons only (Playwright)

### Legacy Code (do not use)
- `backend/app/api/tasks.py` — superseded by `protected_tasks.py`
- `backend/app/api/auth.py` — superseded by `auth_routes.py`
- `backend/app/api/metrics.py` — superseded by `protected_metrics.py`
- `backend/app/api/config.py` — superseded by `protected_config.py`

### Next Steps
1. **LLM execution e2e validation** — run full task→run→metrics pipeline with real OpenRouter API
2. **Run tests locally** — `cd backend && pip install -r requirements.txt && pytest tests/ -v`
3. **Generate Alembic migration** — `cd backend && alembic revision --autogenerate -m "initial"`
4. **Frontend E2E tests** — flesh out Playwright test skeletons

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IceThunder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
