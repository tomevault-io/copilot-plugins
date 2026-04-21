## nexa-ecommerce

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nexa Ecommerce is a multi-role marketplace platform with:
- **`backend/`** — Node.js/Express REST API (ESM, port 4001)
- **`Ecommerce/`** — React 18 + Vite + TypeScript frontend (port 3000 dev)
- **`Mobile/`** — Mobile app (currently empty)
- **`docker-compose.yml`** — Production deployment
- **`docker-compose.dev.yml`** — Development overrides (extends prod compose)

## Development Commands

### Backend
```bash
cd backend
npm run dev          # nodemon src/server.js (hot reload)
npm start            # node server.js (production)
npm test             # jest

# Database migrations (uses config/config.cjs)
npx sequelize-cli db:migrate
npx sequelize-cli db:migrate:undo
npx sequelize-cli db:seed:all
```

### Frontend
```bash
cd Ecommerce
npm run dev          # vite dev server
npm run build        # production build
npm run preview      # preview production build
```

### Docker
```bash
# Production
docker compose up --build

# Development (extends docker-compose.yml)
docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build
```

## Architecture

### Backend (`backend/src/`)
- **`server.js`** — Entrypoint, starts Express on `PORT` env var (default 4001)
- **`app.js`** — Express app setup: helmet, CORS, rate limiting, compression, route registration, DB init on startup
- **`routes/`** — One file per domain, all prefixed `/api/<domain>`
- **`controllers/`** — Request handlers, named `<domain>Controller.js`
- **`models/`** — Sequelize models auto-loaded by `models/index.js`; uses `DATABASE_URL` env var
- **`middlewares/authMiddleware.js`** — JWT auth via `accessToken` cookie; `requireAuth` + `requireAdmin` guards
- **`middlewares/permissionMiddleware.js`** — Role-based permission checks
- **`migrations/`** — Sequelize CLI migrations (`.cjs` for older ones, `.js` for newer ones)
- **`config/config.cjs`** — Sequelize CLI config; detects Docker via `DOCKER_ENV=true` and sets host to `db`

The backend uses **ESM** (`"type": "module"`) but Sequelize CLI config and older migrations use **CommonJS** (`.cjs`).

In **development**, `sequelize.sync()` is available but disabled by default; run migrations manually.
In **production** (Docker), `npx sequelize-cli db:migrate` runs automatically on container start before `npm start`.

### Frontend (`Ecommerce/src/`)
- **`App.tsx`** — Root routes; all lazy-loaded. Uses `react-router-dom` v7.
- **`components/api.ts`** — Axios instance (`baseURL: http://localhost:4001/api`, `withCredentials: true`). Handles 401 auto-refresh via `/auth/refresh` endpoint.
- **`components/ProductContext.tsx`** — Global product state provider wrapping entire app
- **`components/DashboardLayout.tsx`** — Authenticated layout shell wrapping all dashboard routes
- **`page/`** — Feature pages grouped by role: `Admin/`, `Vendeur/`, `Spécialiste/`, `Fournisseur/`, `Commande/`, `Retrait/`, `tickets/`, `parrainage/`
- **`components/`** — Shared components + role-specific dashboards
- **`context/LanguageContext.tsx`** — i18n context (i18next + react-i18next)

### User Roles
- **Admin** — Full platform management, users, permissions, specialists
- **Spécialiste** — Manages products catalog and assigned users
- **Vendeur** — Sells products, manages orders, withdrawals
- **Fournisseur** — Supplier, handles pickup and product lists
- **Client** — End customer

### Key Data Models (Sequelize/PostgreSQL)
`User`, `Vendeur`, `Fournisseur`, `Client`, `Pack`, `Produit`, `Variation`, `Commande`, `LigneCommande`, `SousCommande`, `Categorie`, `Tickets`, `TicketsMessage`, `Transaction`, `DemandeRetrait`, `Parrainage`, `Notification`, `Permission`, `Task`, `Pickup`, `Tracking`, `Media`, `MesProduit`

### Authentication
- JWT stored in **httpOnly cookies** (`accessToken`)
- Refresh token flow via `POST /api/auth/refresh`
- Frontend axios interceptor handles auto-refresh on 401

## Environment Variables

Backend requires a `.env` file in `backend/`:
- `DATABASE_URL` — PostgreSQL connection string
- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` — Used by Sequelize CLI config
- `JWT_SECRET` — Required
- `JWT_EXPIRES_IN`, `BCRYPT_SALT`
- `DOCKER_ENV=true` — Forces DB host to `db` (Docker service name)
- `FRONTEND_URL` — Allowed CORS origin
- Cloudinary, Twilio, Nodemailer credentials as needed

Docker compose requires `.env` at repo root with the same postgres vars + `JWT_SECRET`.

## Important Notes

- The frontend `api.ts` hardcodes `http://localhost:4001/api` as baseURL — in production behind a reverse proxy this needs to be adjusted via env/vite config.
- The Ecommerce `Dockerfile` copies build output to `/app/build` but Vite outputs to `dist/` by default — verify `vite.config.ts` `build.outDir` if the Docker build fails.
- Code comments and variable names mix French and English throughout the codebase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ranim-sioud) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
