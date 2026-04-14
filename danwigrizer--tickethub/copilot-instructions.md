## tickethub

> Configurable fake ticket marketplace designed to test how AI models parse and interact with web content.

# TicketHub

Configurable fake ticket marketplace designed to test how AI models parse and interact with web content.

## Architecture

- **Frontend**: Next.js 14 (React, TypeScript, Tailwind CSS) — `frontend/`
- **Backend**: Express.js (JavaScript) — `backend/`
- **Monorepo**: Root `package.json` orchestrates both via `concurrently`
- **No database**: In-memory data with JSON file-based configuration (`config/active.json`)

## Common Commands

```bash
# Install all dependencies
npm run install:all

# Run both frontend + backend in dev mode
npm run dev

# Run individually
npm run dev:frontend    # Next.js on port 3000
npm run dev:backend     # Express on port 3001

# Build
npm run build           # Installs all deps + builds frontend

# Docker
docker-compose up --build
```

## Project Structure

```
frontend/
  app/                  # Next.js app router pages
    page.tsx            # Home — event listings
    events/[id]/        # Event detail + listings table
    listings/[id]/      # Resale listing detail + purchase
    cart/               # Shopping cart + checkout
    admin/              # Admin config panel
      experiments/      # Experiment list + creation
        [id]/           # Experiment detail + results
  lib/api.ts            # API URL configuration
backend/
  server.js             # Express server (all routes, data, logic)
  experiments.js        # Experiment CRUD, assignment, results
config/
  scenarios/            # 10 preset test scenarios (JSON)
  active.json           # Current config (gitignored)
```

## Environment Variables

**Backend** (`backend/.env`):
- `PORT` — default 3001
- `CORS_ORIGIN` — allowed origin (use `*` for dev)
- `NODE_ENV` — production/development

**Frontend** (`frontend/.env.local`):
- `NEXT_PUBLIC_API_URL` — backend API URL (e.g., `http://localhost:3001/api`)

## Key API Endpoints

- `GET /api/events` — list events
- `GET /api/events/:id` — event detail
- `GET /api/events/:id/listings` — event's resale listings
- `GET /api/listings/:id` — single listing
- `GET /api/config` / `POST /api/config` — read/write configuration
- `GET /api/scenarios` / `POST /api/scenarios/load` — preset scenarios
- `GET /api/cart` / `POST /api/cart` / `DELETE /api/cart/:id` — session-scoped cart ops
- `GET /api/experiments` / `POST /api/experiments` — list/create experiments
- `GET /api/experiments/:id` / `PATCH /api/experiments/:id` / `DELETE /api/experiments/:id` — manage experiment
- `GET /api/experiments/:id/results` — experiment results
- `GET /health` — health check

## Deployment

- **Railway**: Single-volume architecture with symlinks via `entrypoint.sh`
- **Docker**: Multi-stage builds, `docker-compose.yml` for local orchestration
- See `RAILWAY_DEPLOYMENT.md` and `DEPLOYMENT.md` for details

## Code Conventions

- Frontend: TypeScript with strict mode, path aliases (`@/*`)
- Backend: Plain JavaScript (no TypeScript)
- Styling: Tailwind CSS utility classes
- Next.js standalone output mode with security headers enabled

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danwigrizer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danwigrizer)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
