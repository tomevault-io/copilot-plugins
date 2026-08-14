## planificador

> Docker Compose project with decoupled architecture: **React 18 frontend** + **Node.js/Express backend** + **SQLite** database, served via **Nginx** reverse proxy.

# AGENTS.md - Planificador de Recursos y Presupuestos

## Project Overview
Docker Compose project with decoupled architecture: **React 18 frontend** + **Node.js/Express backend** + **SQLite** database, served via **Nginx** reverse proxy.

## Architecture & Data Model
- **Frontend** (`planificador-web`): React 18 + React Router 6 + Tailwind CSS + Recharts + React Joyride (guided tour)
- **Backend** (`planificador-api`): Node.js + Express + SQLite3 + Multer (PDF uploads)
- **Database**: SQLite (`backend/data/planificador.sqlite`)
  - Tables: `budgets`, `executed_budgets`, `personnel`, `accepted_budgets`, `users`, `roles`, `user_roles`, `sessions`
  - Key fields in `budgets` / `executed_budgets`: `client`, `notes`, `ticketRef`, `pdfFilename`, `pdfOriginalName`, `laborBreakdown`, `assignedPersonnel`
- **Proxy**: Nginx proxies `/api` → backend, serves SPA fallback
- **Production**: Runs behind Traefik (external network `traefik_default`), no exposed ports
- **Local dev**: Uses `docker-compose.override.yml` (exposes port 8086, gitignored)

## Key Commands

### Local Development (Docker)
```bash
# Build and start
docker compose up --build -d

# Status / logs
docker compose ps
docker compose logs -f planificador-api

# Frontend URL: http://localhost:8086
```

### Frontend (local, without Docker)
```bash
npm install
npm start          # dev server on port 3000
npm run build      # production build to build/ (includes iOS 12 Safari polyfill postbuild script)
npm test           # react-scripts test (Jest)
```

### Backend (local, without Docker)
```bash
cd backend
npm install
npm start          # runs on port 3001
npm test           # runs node --test (native test runner)
```

## Key Technical Details & Gotchas
- **Browserslist (iOS 12 Safari support)**: `src/index.js` imports `react-app-polyfill/stable`; postbuild script `scripts/fix-ios12-bundle.js` runs after build.
- **Authentication**: Session-based, HttpOnly cookie `planificador_session`. Roles: `admin`, `editor`, `viewer`.
  - Admin bootstrap: set `BOOTSTRAP_ADMIN_EMAIL` + `BOOTSTRAP_ADMIN_PASSWORD` env vars, or auto-creates `admin@planificador.local` with random password.
- **PDF Storage**: Files stored in `/app/upload/pdfs` (mapped to `/var/lib/contel-planificador/upload` in production).
- **Documentation Maintenance Rule**: When instructed to update documentation, update `release.md`, `context.md`, and `README.md`.

## Recent Changes (v0.14)
- **v0.14**: Added `client` field support across budgets, executed budgets, forms, planning view, and CSV export (with automatic SQLite `ALTER TABLE` migration).
- **v0.13**: Added `notes` field (free-text annotations with 📝 badge) in planning budgets and templates.
- **v0.12**: 3-state acceptance date sorting toggle in accepted bag and planning list; reordered PlanningView UI layout.

---
> Source: [LearningCodePython/planificador](https://github.com/LearningCodePython/planificador) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
