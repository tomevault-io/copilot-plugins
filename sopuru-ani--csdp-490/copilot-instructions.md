## csdp-490

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**LostLink** — a campus lost & found management system with AI-powered item matching and real-time messaging. Full-stack: React (Vite) frontend + FastAPI (Python) backend + Supabase (PostgreSQL + Auth + Storage).

---

## Development Commands

### Frontend (root directory)

```bash
npm run dev        # Start Vite dev server at http://localhost:5173
npm run build      # Production build → dist/
npm run lint       # ESLint check
npm run preview    # Preview production build
npm run host       # Dev server exposed on all interfaces
```

### Backend (`Backend/` directory)

```bash
cd Backend
source venv/Scripts/activate          # Windows (bash); use venv/bin/activate on Linux/Mac
uvicorn main:app --reload             # Start dev server at http://localhost:8000
```

The backend has no dedicated test runner configured. To verify endpoints, run the server and use the auto-generated FastAPI docs at `http://localhost:8000/docs`.

---

## Architecture

### Two-Service Structure

- **Frontend** lives at the repo root (Vite + React 19 + Tailwind v4 + shadcn/ui)
- **Backend** lives in `Backend/` (FastAPI + Uvicorn)
- They communicate via HTTP (`http://localhost:8000` in dev) and WebSocket (`ws://localhost:8000`)
- The frontend API base URL is **hardcoded** in [src/lib/api.js](src/lib/api.js) — update this for production builds

### Frontend Routing

[src/routes/AppRoutes.jsx](src/routes/AppRoutes.jsx) defines all routes. The layout wrapper [src/pages/Overview.jsx](src/pages/Overview.jsx) renders the sidebar + content area for authenticated pages. Public routes (login, signup, home) skip this wrapper.

### API Communication Pattern

All frontend → backend calls go through `apiFetch()` in [src/lib/api.js](src/lib/api.js):
- Uses `credentials: "include"` for cookie-based auth
- Catches HTTP 429 for rate limiting

### Authentication Flow

1. Login → backend sets `access_token` + `refresh_token` as httpOnly cookies
2. Every backend route uses `get_current_user(request)` dependency (reads cookie, validates with Supabase)
3. For WebSocket connections, the frontend fetches the token via `GET /auth/token` then passes it as a URL query param (`?token=...`) — browsers cannot set custom headers on WebSocket connections

### Backend Router Organization

`Backend/main.py` (main FastAPI app, ~1250 lines) includes modular routers from `Backend/routers/`:
- `messaging.py` — WebSocket + REST for conversations/messages
- `users.py` — Profile, password, privacy settings
- `pushsubs.py` — Web push VAPID subscriptions
- `notifications.py` — Push notification dispatch (currently disabled in main)

Two Supabase clients are initialized in `main.py`:
- `auth_supabase` — anon key, used for auth operations
- `db_supabase` — service role key, used for all DB reads/writes

### WebSocket Messaging

- `RoomManager` class in [Backend/routers/messaging.py](Backend/routers/messaging.py) holds in-memory `{conversation_id: {user_id: WebSocket}}` mappings
- Frontend manages connection lifecycle in [src/pages/Messages.jsx](src/pages/Messages.jsx) using `useRef` to prevent reconnection loops
- Message payload from server: `{type: "message", data: {...}}`
- In-memory room state is fine for a single server instance but won't survive restarts

### AI Matching

`GET /items/{item_id}/matches` sends item data + signed image URLs to **Google Gemini 2.5 Flash Lite**. Returns up to 5 candidate matches scoring ≥ 40. The prompt construction and response parsing are in `main.py`.

### Audit Logging

`log_action(actor_id, action, target_type, target_id, details)` in `main.py` — called after sensitive admin/user operations. Writes to the `audit_logs` table.

---

## Environment Variables

### Frontend (`.env` at repo root)

| Variable | Purpose |
|---|---|
| `VITE_SUPABASE_URL` | Supabase project URL |
| `VITE_SUPABASE_ANON_KEY` | Public JWT for client-side Supabase |
| `VITE_API_WS_URL` | WebSocket backend base URL |

### Backend (`Backend/.env`)

| Variable | Purpose |
|---|---|
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_ANON_KEY` | Auth operations |
| `SUPABASE_SERVICE_ROLE_KEY` | Unrestricted DB access |
| `FRONTEND_URL` | CORS allowlist |
| `GEMINI_API_KEY` | Google Gemini for AI matching |
| `GMAIL_ADDRESS` / `GMAIL_APP_PASSWORD` | Email sending |
| `VAPID_PUBLIC_KEY` / `VAPID_PRIVATE_KEY` / `VAPID_CLAIMS_EMAIL` | Web push notifications |
| `ENV` | Set to `"production"` to enable secure cookies (`SameSite=None; Secure`) |

---

## Deployment

- **Frontend → Vercel**: `vercel.json` rewrites all routes to `index.html` for SPA routing. Before deploying, update the hardcoded API URL in `src/lib/api.js` and set `VITE_API_WS_URL` to the production backend URL.
- **Backend → Render** (or any ASGI host): `uvicorn main:app --host 0.0.0.0 --port 8000`. Set `ENV=production` so cookies are sent cross-origin.
- **PWA**: Service worker at [src/sw.js](src/sw.js) uses Workbox `injectManifest` strategy. Handles offline fallback and push notification display.

---

## Key Constraints

- **Rate limiting** (slowapi): varies by endpoint — signup is 5/min, item creation is 20/min, abuse reports are 5/min. Tests that hit these routes repeatedly will get 429s.
- **No migrations**: Supabase manages the schema. Table definitions are not in this repo.
- **Image storage**: Supabase Storage bucket `item-images`, path format `{user_id}/{date}/{uuid}_{filename}`. Signed URLs expire after 1 hour.
- **Admin role**: Checked via `is_admin` field on the `users` table row, not via Supabase Auth roles.

---
> Source: [sopuru-ani/csdp_490](https://github.com/sopuru-ani/csdp_490) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
