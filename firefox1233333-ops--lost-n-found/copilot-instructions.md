## lost-n-found

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Lost & Found Management System for a college campus, built with the MERN stack. Monorepo with two independent packages: `client/` (React/Vite frontend) and `server/` (Express/MongoDB backend). There is no root-level `package.json`; each package is managed separately.

## Build & Run Commands

### Server (Express backend)
```
cd server
npm install          # install dependencies
npm run dev          # start with nodemon (hot-reload), runs on port 5000
npm start            # start without hot-reload
npm run seed         # seed database with sample users and items (scripts/seed.js)
```

### Client (React frontend)
```
cd client
npm install          # install dependencies
npm run dev          # start Vite dev server, runs on http://localhost:5173
npm run build        # production build
npm run lint         # run ESLint
npm run preview      # preview production build
```

Both servers must run simultaneously in separate terminals. There is no test suite configured for either package.

### Environment Variables

Server requires `server/.env` with:
- `MONGO_URI` – MongoDB connection string (local or Atlas)
- `JWT_SECRET` – secret for signing JWTs (note: historically misspelled as `JWT_SECRECT` — must be `JWT_SECRET`)
- `PORT` – defaults to 5000

Client optionally uses `client/.env` with:
- `VITE_API_URL` – API base URL (defaults to `http://localhost:5000/api`)

## Architecture

### Three User Roles

The system has three roles with different access levels: `user`, `security`, `admin`. Roles are stored on the User model and enforced by middleware (`server/middleware/auth.js`) via composable guards: `requireAuth`, `requireAdmin`, `requireSecurity`, `requireAdminOrSecurity`, `optionalAuth`.

- **user** – can report lost/found items, browse, claim found items, view own reports
- **security** – can do everything a user can, plus update item status and add found items via a dedicated security dashboard
- **admin** – full access including deleting items, managing users (role changes), and the admin dashboard

### Backend (`server/`)

CommonJS modules (`"type": "commonjs"`). Entry point is `server.js`.

- `config/db.js` – Mongoose connection setup
- `models/User.js` – User schema (name, email, password, role enum)
- `models/Item.js` – Item schema with `type` (lost/found), `status` (Lost/Found/At Security/Returned), `claimedBy`/`claimStatus` for the claim workflow
- `routes/authRoutes.js` – registration and login; JWT issued with 7-day expiry
- `routes/itemRoutes.js` – CRUD for items, claim endpoint, filtering via query params
- `routes/userRoutes.js` – admin-only user listing and role updates
- `middleware/auth.js` – JWT verification and role-based guards
- `scripts/seed.js` – seeds 3 users (1 admin, 2 regular) and 6 sample items

API routes are mounted at `/api/auth`, `/api/items`, `/api/users`.

### Frontend (`client/`)

ES modules, React 19 with Vite. No TypeScript — all `.jsx` files.

- `main.jsx` – mounts `<BrowserRouter>` wrapping `<App />`
- `App.jsx` – defines all routes and role-based route guards (`ProtectedRoute`, `AdminRoute`, `SecurityRoute`)
- `context/AuthContext.jsx` – provides `user`, `login`, `register`, `logout`, `isAdmin`, `isSecurity` via React Context. Auth state persisted in `localStorage` (`token` and `user` keys).
- `services/api.js` – centralized API client using `fetch`; auto-attaches Bearer token from localStorage. Exports `authAPI`, `itemsAPI`, `usersAPI`.
- `components/RoleBasedLayout.jsx` – switches the entire app shell (navigation + layout) based on role: `AdminLayout`, `SecurityLayout`, or `AppLayout` (for regular users/guests)
- `pages/admin/` – admin-specific pages (dashboard, user management)
- `pages/security/` – security officer pages (dashboard, add found items)

### Key Patterns

- Auth token is passed as `Authorization: Bearer <token>` on every API request
- Item `type` field distinguishes lost vs found reports; `status` tracks the lifecycle (Lost → Found → At Security → Returned)
- Items can be claimed by users (`claimedBy`/`claimStatus` fields); only found items are claimable
- Item categories are a fixed enum: Documents, Electronics, Accessories, Clothing, Other
- Filtering is done server-side via query parameters on `GET /api/items`
- CSS is per-component (no CSS-in-JS or utility framework)

---
> Source: [firefox1233333-ops/lost_n_found](https://github.com/firefox1233333-ops/lost_n_found) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
