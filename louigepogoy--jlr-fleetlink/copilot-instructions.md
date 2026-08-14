## jlr-fleetlink

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

JLR Fleetlink — a vehicle rental platform (Cebu-focused) with role-based dashboards (Customer, Owner, Admin), online payments (simulated GCash/Card), and commission-based platform earnings.

**Only two directories make up the running application:**
- `backend/` — Express.js REST API
- `frontend/` — Next.js 16 (App Router) frontend, TypeScript + Tailwind

`client/` and `server/` at the repo root are unused scaffold directories (default `create-next-app` and empty Express init) accidentally committed in the "first push" commit. They are not wired to anything and not part of the app — don't edit them or treat them as alternate frontend/backend.

## Commands

### Backend (`backend/`)
```bash
npm run dev        # nodemon src/index.js — API on http://localhost:5000
npm start           # node src/index.js (production)
npm run db:setup    # apply schema.sql + seed demo users (scripts/setup-database.js)
npm run seed         # re-seed/reset demo user passwords only (scripts/seed-users.js)
```
No test runner or lint config exists in `backend/`.

### Frontend (`frontend/`)
```bash
npm run dev     # next dev — app on http://localhost:3000
npm run build
npm run lint    # eslint
```
No test runner is configured in `frontend/`.

There is no root-level build; run backend and frontend independently, each in its own terminal.

### Database
Postgres (Neon recommended). There is no generic migration runner — `backend/scripts/setup-database.js` hardcodes an explicit list of migration file paths and runs them in order (only when the `users` table already exists; a brand-new database instead gets the full `schema.sql`). When adding a schema change: create a new numbered file under `backend/database/migrations/`, add a `const ..Path = path.join(...)` + `runSqlFile(...)` call for it in `setup-database.js`'s "tables already exist" branch, and also update `schema.sql` so fresh installs match. Forgetting to wire a new migration into `setup-database.js` means `npm run db:setup` silently won't apply it to an existing (e.g. Neon) database.

## Architecture

### Backend request flow
Plain layered Express app, no ORM — raw SQL via `pg` (`backend/src/config/db.js` exports a pooled `query(text, params)` helper). Pattern per feature: `routes/*.routes.js` → `middleware/auth.js` (`authenticate`, `authorize(...roles)`) → `controllers/*Controller.js` (SQL + response shaping) → occasionally `services/` (e.g. `paymentService.js` simulates GCash/Card processing) or `utils/`.

All routes are mounted in `backend/src/index.js` under `/api/<resource>` (auth, users, vehicles, bookings, payments, transactions, commission, notifications, subscriptions, reports). CORS is an explicit allow-list of localhost origins plus `FRONTEND_URL` — update `allowedOrigins` in `index.js` when adding new frontend origins.

Auth: JWT bearer tokens (`Authorization: Bearer <token>`), verified in `authenticate` middleware which also re-fetches the user row and enforces `is_active` and `approval_status` (non-admins with `approval_status !== 'approved'` are rejected — this is the driver's-license/selfie admin-approval gate described in the README). `authorize('admin', 'owner', ...)` gates by `role`.

Uploads: `multer` writes license/selfie images to `backend/uploads/`, served statically at `/uploads`.

### Database
`backend/database/schema.sql` is the source of truth (Postgres enums for `user_role`, `vehicle_status`, `booking_status`, `payment_status`, `payment_method`, `transaction_type`, `notification_type`, `approval_status`, `report_status`). Incremental schema changes since the initial schema live as one-off files in `backend/database/migrations/` (not auto-applied) and as top-level `migration_*.sql` files in `backend/database/`. When changing the schema, add a new numbered file to `migrations/` rather than editing `schema.sql` in place, and update `schema.sql` to match so a fresh `db:setup` stays consistent.

Notable domain constraint: `vehicles.city` is restricted via a CHECK constraint to a fixed list of Cebu cities/municipalities — the platform is scoped to Cebu.

### Frontend
Next.js App Router under `frontend/src/app/`, routes grouped by role: `dashboard/customer/*`, `dashboard/owner/*`, `dashboard/admin/*`, plus public `vehicles/`, `auth/login`, `auth/register`. Shared UI in `frontend/src/components/` (grouped by feature: `booking/`, `payment/`, `vehicles/`, `dashboard/`, `home/`, `layout/`, `reports/`, `receipt/`, `ui/`).

State: Zustand stores in `frontend/src/store/` (`authStore.ts`, `themeStore.ts`) — not Redux/Context.

API access: all requests go through the single Axios instance in `frontend/src/lib/api.ts`, which attaches the JWT from `localStorage` and redirects to `/auth/login` on a 401. Use this instance rather than raw `fetch`/`axios` for new API calls so auth headers and 401 handling stay consistent.

`NEXT_PUBLIC_API_URL` (defaults to `http://localhost:5000/api`) points the frontend at the backend.

### Payments
Simulated in `backend/src/services/paymentService.js` — no real GCash/Stripe integration yet (README flags this as the spot to swap in a real PSP for production). Commission is deducted automatically per the `platform_settings.commission_percentage` row; admins can change the rate via `PUT /api/commission`.

---
> Source: [Louigepogoy/JLR-FLEETLINK-](https://github.com/Louigepogoy/JLR-FLEETLINK-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
