## wm-canteen-tool

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- **Dev server:** `npm run dev` (uses tsx watch for hot-reload)
- **Build:** `npm run build` (TypeScript → `dist/`)
- **Start (prod):** `npm start` (runs `dist/server.js`)
- **Local database:** `docker compose up -d` (PostgreSQL on localhost:5432)

No test framework is configured.

## Architecture

Single-file Express + Socket.IO server (`server.ts`) with vanilla HTML/CSS/JS frontend pages in `public/`.

### Backend (`server.ts`)

- **Database:** Raw SQL via `pg.Pool` — no ORM. Helper functions `query()` and `queryOne()` wrap `pool.query()`.
- **Schema init:** `initDb()` runs `CREATE TABLE IF NOT EXISTS` on startup and seeds example items if the `items` table is empty. Tables: `items`, `orders`, `order_items`, `meta` (key-value for order counter).
- **Auth:** PIN-based with three roles (`order`, `kitchen`, `admin`). PINs set via env vars. Sessions use `cookie-session`. Admin role has access to all views.
- **Realtime:** Socket.IO emits events (`new-order`, `order-done`, `item-updated`, `item-deleted`) to all connected clients. Socket connections are authenticated via the same cookie session.
- **Database flexibility:** Connects to local PostgreSQL by default; supports Supabase by detecting "supabase" in `DATABASE_URL` to enable SSL.

### Frontend (`public/`)

- Separate HTML/JS/CSS per view: `login`, `order`, `kitchen`, `admin`, plus shared `index.html` (landing redirect).
- `config.js` fetches `/api/config` for locale/currency formatting — imported by other pages.
- `theme.js` handles dark/light mode toggle, persisted in `localStorage`. All views use CSS variables from `shared.css`; the `[data-theme="dark"]` selector overrides them.
- Each view connects its own Socket.IO client for realtime updates.
- No build step, no bundler, no framework — plain browser JS with `fetch()` and DOM manipulation.
- Login page supports `?redirect=` query param for re-auth flows (e.g. switching to a higher-privilege view).

### Deployment

- **Render** via `render.yaml` + `Dockerfile` — auto-deploys on push to `main`.
- **Supabase** for PostgreSQL — must use the **Session Pooler** connection string (IPv4-compatible). Direct connections require paid IPv4 add-on.
- `Dockerfile` includes `--dns-result-order=ipv4first` and `gai.conf` to force IPv4 resolution.
- Dependabot configured for npm and Docker updates (`.github/dependabot.yml`).

### Environment Variables

Configured in `.env` (see `.env.example`): `DATABASE_URL`, `ORDER_PIN`, `KITCHEN_PIN`, `ADMIN_PIN`, `SESSION_SECRET`, `PORT`, `LOCALE`, `CURRENCY`.

---
> Source: [martin-weber-1997/wm-canteen-tool](https://github.com/martin-weber-1997/wm-canteen-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
