## sporty

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Sporty is a multi-sport fantasy league platform (Football/Soccer, NBA Basketball, Cricket). Users create or join leagues, build squads under a budget, set weekly lineups, and earn points from real match performance. Leagues can be single-sport or mixed (football + basketball in one squad).

This is a **monorepo** with two independently-deployed apps plus supporting data:

- `Sporty_Backend/` — FastAPI API + Celery/APScheduler workers + data ingestion. **Has its own detailed `Sporty_Backend/CLAUDE.md` — read it before working in the backend.**
- `sporty-frontend/` — Next.js 16 (App Router) + React 19 + TypeScript UI. Has `sporty-frontend/CLAUDE.md` and `AGENTS.md` covering frontend coding conventions (Mantine + Tailwind, services/store/Zod separation).
- `EPL/`, `basketball/` — raw CSV stat datasets used by backend seeders/ingestion.
- `graphify-out/`, `merge_chunks.py` — local knowledge-graph tooling artifacts (untracked, not app code).
- `docs/`, `diagrams/` — architecture writeups and diagrams spanning both apps.

## Commands

### Full local stack (recommended)
```bash
docker compose up    # postgres + redis + API (:8000) + celery worker/beat + frontend (:3000)
```
Root `docker-compose.yml` uses its own local DB with dev credentials (never
the production `.env`) and leaves a fresh stack migrated + seeded.

### Backend (`Sporty_Backend/`)
See `Sporty_Backend/CLAUDE.md` for the full set. Quick reference (venv at `Sporty_Backend/venv/`, no `pyproject.toml`):
```bash
venv/bin/uvicorn app.main:app --reload --port 8000   # API (:8000 = what the frontend proxy expects)
venv/bin/python -m pytest                             # tests (pip install -r requirements-dev.txt first)
venv/bin/alembic upgrade head                         # migrations
venv/bin/celery -A app.core.celery_app.celery_app worker --loglevel=INFO
```

### Frontend (`sporty-frontend/`)
Package manager is **Yarn 4** (`packageManager: yarn@4.15.0`); use `yarn`, not npm.
```bash
yarn dev          # dev server on :3000 (proxies /api/* → backend)
yarn build        # next build (CI runs this on every push)
yarn start        # serve production build
yarn lint         # eslint
yarn test         # vitest unit tests (src/**/*.test.ts), runs in CI
yarn test:e2e     # Playwright smoke flows (e2e/) — needs `docker compose up` first
```
Deployment: frontend on **Vercel**, backend on **Render** (Docker). There is
no frontend Docker image and no Cloudflare/wrangler path.

## Cross-app integration (the part that requires reading both sides)

**Auth & API contract.** The frontend talks to the backend entirely over `/api/v1` using **httpOnly-cookie JWT + CSRF double-submit**. There is no token in JS — `src/api/auth-api-client.ts` holds the authenticated Axios instance (auto-refreshes on 401), `src/api/public-api-client.ts` manages the in-memory CSRF token. State-changing requests must carry `X-CSRF-Token`. The backend's CSRF/CORS/cookie middleware must match this; CORS origins and `COOKIE_DOMAIN` are environment-specific on the backend.

**Dev cross-origin handling.** `next.config.ts` rewrites `/api/:path*` to `BACKEND_SERVER_URL` (default `http://localhost:8000`) so cookies are same-origin in dev (avoids `SameSite=Lax` blocking POST/PUT/etc.). `NEXT_PUBLIC_API_URL` (e.g. `/api/v1`) is the base path the Axios clients prepend.

**Endpoint registry.** Every backend endpoint the frontend calls is registered centrally in `src/api/apiPath.ts` (`API_PATHS`). Services consume those constants — URLs are never hard-coded elsewhere. When you add/rename a backend route, update `API_PATHS` and the corresponding service.

## Frontend architecture (not covered by the frontend CLAUDE.md, which is conventions-only)

Path alias `@/*` → `src/*` (`tsconfig.json`). Layered data flow is **Backend → services → hooks (React Query) → store/UI**:

- `src/app/` — App Router, organized into route groups `(auth)`, `(dashboard)`, `(public)`, plus `match/[matchId]`. `next.config.ts` also defines `/league/:id*` → `/leagues/:id*` redirects.
- `src/api/` — the two Axios instances + `apiPath.ts`.
- `src/services/` — typed API call functions per domain (`LeagueService`, `TeamService`, `PlayerService`, `ScoringService`, `OptimizationService`, `UserService`, …). UI never calls Axios directly.
- `src/hooks/` — React Query wrappers (`hooks/api/useApiQuery.ts`, `useApiMutation.ts`) and domain hooks (`leagues`, `my-team`, `players`, `scoring`, …) that call services.
- `src/store/` — Zustand (e.g. `matchStore.ts`).
- `src/lib/` — cross-cutting client utilities: `realtimeApi.ts` / `socket.ts` (WebSocket/SSE for live scoring), `league-lifecycle.ts`, `storage.*` (typed localStorage/session helpers), `route.config.ts`, `sanitize.ts`, `validations.ts`.
- `src/features/` — feature modules (`create-league`, `create-team`, `my-team`, `transfers`, `leagues`, …) composing components + hooks.

The realtime/live-scoring frontend (`src/lib/realtimeApi.ts`, `socket.ts`, `components/live/`) consumes the backend's live-data → Redis pub/sub → WebSocket/SSE chain. Football live data comes from API-Football polling (2026-07; the SportyDataFeeder push path is commented out in `app/main.py` but kept re-enableable — see `Sporty_Backend/CLAUDE.md`). (A Kafka pipeline also exists but is dormant — flag off, deps uninstalled — kept for a future real-matches API.)

## Backend architecture (summary — full detail in `Sporty_Backend/CLAUDE.md`)

Vertical-slice modules under `app/` (each with `models/router/services/schemas`). Two DB sessions: **sync** (`app/database.py`, psycopg2) for most routes/jobs, **async** (`app/core/database.py`, asyncpg) for realtime only. Key conventions: top-level service entry points own and commit their transaction, helpers called by other services never commit (see the backend CLAUDE.md for the full rule); all model modules are imported in `app/main.py` before routers so SQLAlchemy resolves relationships. Sports are normalized behind `ISportAdapter` (`app/adapters/`); squad rules in `app/league/sportConfigs.py`. Scoring has a batch/gameweek layer (`app/services/scoring/`) and a live layer applying the same rules to feeder events (`app/services/feed_scoring.py`). Two active background systems: APScheduler (in-process cron, Redis-locked per job) and Celery + Beat (sync/polling/pricing); a third (Kafka realtime) is dormant. Squad auto-pick uses PuLP ILP (`app/services/optimization/`).

---
> Source: [sam0690/Sporty](https://github.com/sam0690/Sporty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
