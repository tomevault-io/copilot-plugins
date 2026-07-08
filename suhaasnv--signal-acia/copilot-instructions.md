## signal-acia

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Signal (ACIA) — a competitive-intelligence tool that scrapes a competitor's pricing page, diffs it against a stored baseline, and generates an AI insight only when the change is material. Frontend and backend are two separate npm projects in one repo (`/` and `/server`).

## Commands

Frontend (run from repo root):
```bash
npm run dev          # Vite dev server → http://localhost:5173
npm run build         # production build
npm run build:dev     # development-mode build
npm run lint           # eslint .
npm run test           # vitest run (single run)
npm run test:watch     # vitest watch mode
```
Run a single test file: `npx vitest run src/test/example.test.ts`

Backend (run from `server/`):
```bash
npm run dev    # node --watch server.js → http://localhost:3001
npm start       # node server.js
```

Run both together from root: `npm run dev:full` (concurrently runs frontend + backend).

There is no backend test runner configured — `server/test-*.js` files are standalone manual scripts (`node test-endpoints.js`, etc.), not part of an automated suite.

Before committing, per user-global standards: `npm run build` must show a clean compile, and any layout/animation change must be checked at 375px / 768px / 1280px.

## Architecture

### Two independent servers, one Railway Postgres backend
- **Frontend** (`src/`): React 18 + TypeScript + Vite + Tailwind + shadcn/ui. Owns no third-party keys — all auth and data operations go through the Express API via `src/lib/api.ts`. Deployed on Railway as its own service, served statically by `serve.js` (Express + SPA fallback) after `vite build`.
- **Backend** (`server/`): Express API. Owns all third-party API keys (OpenAI, Bright Data, ActionBook, Acontext) and the Postgres connection — the frontend never calls these directly. Auth is custom JWT + bcrypt (`server/src/services/auth.service.js`, `server/src/middleware/auth.middleware.js`); there is no external auth provider.
- **Database**: Railway Postgres, accessed via `server/src/services/db.service.js` (raw `pg` queries, no ORM). Schema lives in `db/schema.sql`. One row per user in `competitors`/`reports`-style tables is overwritten on each scan — there is no history table by design (see "Cost Efficiency First" below). Every query is scoped by `user_id` at the query layer (no RLS, since there's no separate DB-level auth system).

### Request flow: `POST /api/scan`
This is the core flow of the app; touching any of these files affects the others:
1. `server/src/routes/scan.routes.js` → `server/src/controllers/scan.controller.js`
2. `services/brightdata.service.js` — scrapes the competitor URL. Tries **MCP → Proxy → Direct fetch** in order, with retries/backoff, depending on which credentials are configured.
3. `services/actionbook.service.js` — fallback only, triggered when the parser finds fewer than 2 pricing tiers (e.g. user gave a homepage URL, or pricing is JS-rendered). Uses an autonomous web agent to navigate to the pricing page.
4. `services/parser.service.js` — turns raw HTML into a deterministic JSON schema (`{ tiers: [{ name, price, features }] }`).
5. `services/acontext.service.js` — fetches/stores the per-user baseline snapshot (`get/setLatestSnapshot`). Falls back to an in-memory `Map` if no API key is set — **so with no `ACONTEXT_API_KEY`, "baseline memory" does not survive a server restart.**
6. `services/diff.service.js` — pure-code delta engine, old snapshot vs new snapshot.
7. `services/openai.service.js` — called **only** if the delta crosses a 5% significance threshold, using `OPENAI_MODEL` (default `gpt-4o-mini`). Raw HTML/pages are never sent to the LLM, only the structured delta.
8. `services/db.service.js` — persists the new snapshot + insight, overwriting the previous row (no history).

Cost-first design principle driving all of the above: expensive calls (scraping, LLM) only happen on explicit user action, and the OpenAI call specifically is skipped whenever the diff is below threshold. Don't add polling, cron, or auto-retry loops — this is intentionally a manual, synchronous, single-shot pipeline (see `docs/architecture.md`).

### Auth flow
- `POST /api/auth/register` / `/login` (`server/src/services/auth.service.js`) — bcrypt-hash/verify against `users.password_hash`, sign a 7-day JWT with `JWT_SECRET`.
- `server/src/middleware/auth.middleware.js` — verifies the Bearer JWT on every protected route, attaches `req.user = { id, email }`.
- `POST /api/profile` (`profile.controller.js`) — creates the onboarding profile row; called from `src/pages/Onboarding.tsx` after registration since there's no separate signup-vs-onboarding step server-side.
- No email verification or password reset flow exists yet — registration logs the user in immediately. No Google/SSO login — email+password only.

### Frontend structure
- `src/lib/api.ts` — single `authFetch` wrapper that attaches a JWT (read from `localStorage`, key `signal_auth_token`) to every backend call; all backend calls go through the `api` object here, not raw `fetch`.
- `src/contexts/AuthContext.tsx` + `src/components/ProtectedRoute.tsx` — auth/session state and route guarding. On mount, validates the stored token via `GET /api/auth/me` before trusting it.
- `src/pages/` — route-level screens: `Landing` → `Register`/auth → `Onboarding` (add competitor) → `Dashboard`/`DashboardLayout` (with `dashboard/DashboardTab.tsx` and `dashboard/ReportsTab.tsx`) → `Processing` (live scan trace) → `Report` (delta + insight detail).
- `src/components/ui/` — shadcn/ui primitives (generated, treat as vendored — avoid hand-editing unless necessary).

### Environment variables
Full reference in `.env.example`. `DATABASE_URL`, `JWT_SECRET`, and `OPENAI_API_KEY` are required; everything else (Bright Data, ActionBook, Acontext) is optional and has a documented fallback (Bright Data → direct fetch, Acontext → in-memory map). When debugging "why didn't X service get called," check which env vars are actually set before assuming a code bug.

## Documentation

- [Architecture](docs/architecture.md) — original MVP design doc; describes the intended single-competitor, single-snapshot, no-history scope and the explicit list of features left out (multi-tenant, timeline history, scheduling, polling).
- [README.md](README.md) — live demo links, setup, API reference, tech stack.

---
> Source: [SuhaasNv/signal-acia](https://github.com/SuhaasNv/signal-acia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
