## trading-hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Frontend (Next.js) — uses **pnpm**
```bash
cd frontend
pnpm dev          # Dev server with Turbopack (port 3000)
pnpm build        # Production build
pnpm lint         # ESLint
pnpm type-check   # TypeScript type check (tsc --noEmit)
```

### Backend (Flask)
```bash
cd backend
source .venv/bin/activate
python run.py     # Dev server (port 5000, debug mode)

# Database migrations
flask db init
flask db migrate -m "description"
flask db upgrade
```

### Full Stack (Docker)
```bash
docker compose up         # Start all services
docker compose up -d      # Detached
docker compose down       # Stop all
```

### ML Module
```bash
cd ml
pip install -r requirements.txt
python -m pytest          # (no tests yet)
```

## Architecture

### Request Flow
Browser → Next.js (port 3000) → `/api/*` rewrites → Flask (port 5000) → PostgreSQL/Redis

The `next.config.ts` rewrites all `/api/*` requests to `${NEXT_PUBLIC_API_URL}/api/*`, so frontend code calls `/api/...` without hardcoding the backend URL.

### Frontend Structure
- **Route groups**: `(auth)/` for login/register, `(dashboard)/` for protected pages
- **Middleware** (`src/middleware.ts`): Edge middleware reads `auth-token` cookie to guard `/dashboard/*` routes
- **State**: Zustand for client state (`store/auth.ts`, `store/market.ts`), TanStack Query for server state
- **API client** (`lib/api/client.ts`): Axios instance; modules in `lib/api/{auth,market,portfolio,ai}.ts`
- **Auth store** (`store/auth.ts`): `login()` writes both `localStorage` and `auth-token` cookie; `logout()` clears both

### Backend Structure
- **App factory** (`app/__init__.py`): `create_app(config_name)` wires blueprints, JWT, SQLAlchemy, CORS, Celery
- **Blueprints**: `auth`, `market`, `portfolio`, `trading`, `ai` — each registered with `/api/<name>` prefix
- **Config** (`config.py`): Three configs — `DevelopmentConfig`, `ProductionConfig`, `TestingConfig`
- **AI service** (`services/ai_service.py`): Uses Anthropic Claude (`claude-opus-4-6`) with SSE streaming
- **Market service** (`services/market_service.py`): Currently returns mock data; interface is ready for Tushare/AKShare

### Database Models
`User` → `Portfolio` → `Position` / `Trade` (SQLAlchemy ORM, PostgreSQL in prod, SQLite in-memory for tests)

### ML Module
Standalone Python package. `strategies/base_strategy.py` defines the base class; `strategies/ma_cross.py` is a moving average crossover example. `backtest/engine.py` runs simulations. Does not import from `backend/`.

## Key Conventions

### Color scheme (A股 / Chinese market standard)
- **Bull / price up**: red `#ef4444`
- **Bear / price down**: green `#22c55e`
- Background palette: primary `#0a0e1a`, secondary `#0f1629`, card `#141c2e`

### Responsive layout breakpoints
| Breakpoint | Layout |
|---|---|
| `< md` | No sidebar; top `Header` + bottom `BottomNav` (5 tabs) |
| `md – lg` | Icon-only sidebar (`w-16`) |
| `≥ lg` | Full sidebar (`w-56`) |

Use `.pb-safe` (defined in `globals.css`) for elements that must clear the iOS notch/home indicator.

### Path alias
`@/` maps to `frontend/src/` — use this for all internal imports.

### Environment variables
Copy `.env.example` to `.env`. Required keys: `DB_PASSWORD`, `SECRET_KEY`, `JWT_SECRET_KEY`, `ANTHROPIC_API_KEY`. `NEXT_PUBLIC_API_URL` defaults to `http://localhost:5000`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JferLao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
