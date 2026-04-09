## test

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

News Situation Dashboard — a full-stack app for tracking news articles organized by user-defined "situations" (topics/queries). Includes JWT auth, RSS feed ingestion, and per-situation dashboards with article counts, source counts, and top headlines.

## Architecture

- **Backend**: Python FastAPI app (`backend/app/main.py`) with SQLAlchemy ORM, PostgreSQL via psycopg, JWT auth (python-jose + passlib/bcrypt). Single-file API with all routes in `main.py`.
- **Frontend**: React 18 + Vite SPA (`frontend/src/App.jsx`). Single-component app, no router. Auth state managed via `frontend/src/auth.js` (localStorage).
- **Database**: PostgreSQL 16. Schema in `db/schema.sql` — tables: `app_user`, `situation`, `source`, `article`, `situation_article` (many-to-many), `dashboard_snapshot`, `feed_source`, `feed_article`.
- **RSS Worker**: Background process (`backend/app/worker.py`) using APScheduler, fetches all active `feed_source` RSS feeds every 30 minutes via feedparser. Run as `python -m backend.app.worker`.
- **Reverse Proxy**: Caddy (`infra/caddy/Caddyfile`) — `/api/*` routes to backend:8000, everything else to frontend:80.
- **Autoscaler**: Docker Swarm CPU-based autoscaler for the backend service (`ops/autoscaler/autoscaler.py`).

## Development Commands

### Backend
```bash
pip install -r backend/requirements.txt
cp backend/.env.example backend/.env  # then edit values
psql "$DATABASE_URL" -f db/schema.sql
uvicorn backend.app.main:app --reload
```

### Frontend
```bash
cd frontend && npm install
npm run dev          # Vite dev server on :5173
npm run build        # Production build
```
Set `VITE_API_BASE_URL` — use `/api` behind Caddy or `http://localhost:8000` for direct backend access.

### Docker
```bash
docker compose -f docker-compose.build.yml build   # Build images
docker stack deploy -c docker-stack.ubuntu.yml newsdash  # Deploy (Swarm)
```

## Key Backend Config

Settings loaded via pydantic-settings from `backend/.env` (see `backend/app/config.py`):
- `DATABASE_URL` — SQLAlchemy connection string (default: `postgresql+psycopg://postgres:postgres@localhost:5432/news_dashboard`)
- `JWT_SECRET_KEY` — must change for production
- `ADMIN_EMAIL` — email that auto-gets admin on registration
- `CORS_ORIGINS_CSV` — comma-separated allowed origins
- First registered user automatically becomes admin

## API Endpoint Flow

1. `POST /auth/register` or `POST /auth/login` → get JWT
2. `POST /situations` → create a tracking topic
3. `POST /articles/ingest` (admin) → add articles linked to situations
4. `GET /situations/{id}/articles` → list articles for a situation
5. `GET /situations/{id}/dashboard` → aggregated stats
6. `POST /feed-sources` (admin) → add RSS feeds for background polling
7. `GET /feed-articles` → browse RSS-ingested articles

## Auth Model

JWT Bearer tokens via `Authorization` header. Two roles: regular user (owns situations) and admin (full access). Ownership enforced via `require_situation_access()` — users can only see their own situations, admins see all.

## Deployment Targets

- **Ubuntu VM**: `docker-stack.ubuntu.yml` with Caddy + Cloudflare Tunnel (see `DEPLOY_UBUNTU_CADDY_CLOUDFLARE.md`)
- **Unraid**: `docker-compose.unraid.yml` (see `DEPLOY_UNRAID.md`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/keygol7)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/keygol7)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
