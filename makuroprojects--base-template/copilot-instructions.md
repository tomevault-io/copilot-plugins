## base-template

> Default to using Bun instead of Node.js.

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
- Bun automatically loads .env, so don't use dotenv.

## Server

Elysia.js as the HTTP framework, running on Bun. API routes are in `src/app.ts` (exported as `createApp()`), frontend serving and dev tools are in `src/index.tsx`.

- `src/app.ts` — Elysia app factory with all API routes (auth, admin, logs, presence, hello, health, Google OAuth). Testable via `app.handle(request)`.
- `src/index.tsx` — Server entry. Adds Vite middleware (dev) or static file serving (prod), click-to-source editor integration, audit log rotation, and `.listen()`.
- `src/serve.ts` — Dev entry (`bun --watch src/serve.ts`). Dynamic import workaround for Bun EADDRINUSE race.

## Database

PostgreSQL via Prisma v6. Client generated to `./generated/prisma` (gitignored).

- Schema: `prisma/schema.prisma`
  - `User` (id, name, email, password, role, blocked, timestamps)
  - `Session` (id, token, userId, expiresAt)
  - `AuditLog` (id, userId, action, detail, ip, createdAt)
  - `Ticket` (id, title, description, status, priority, route, reporterId, assigneeId, timestamps, closedAt)
  - `TicketComment` (id, ticketId, authorId, authorTag, body, createdAt)
  - `TicketEvidence` (id, ticketId, kind, url, note, createdAt)
- Enums: `Role` = `USER | QC | ADMIN | SUPER_ADMIN` (default `USER`); `TicketStatus` = `OPEN | IN_PROGRESS | READY_FOR_QC | REOPENED | CLOSED`; `TicketPriority` = `LOW | MEDIUM | HIGH | CRITICAL`
- Client singleton: `src/lib/db.ts` — import `{ prisma }` from here
- Seed: `prisma/seed.ts` — demo users (superadmin, admin, user) with `Bun.password.hash` bcrypt
- Commands: `bun run db:migrate`, `bun run db:seed`, `bun run db:generate`

## Redis

Bun native `Bun.RedisClient` — no external package needed.

- Client singleton: `src/lib/redis.ts` — connects to `REDIS_URL`
- App logs: stored as Redis List (`app:logs`), max 500 entries via `LTRIM`, persists across restart
- App log module: `src/lib/applog.ts` — `appLog(level, message, detail?)`, `getAppLogs(options?)`, `clearAppLogs()`

## Auth

Session-based auth with HttpOnly cookies stored in DB.

- Login: `POST /api/auth/login` — finds user by email, verifies password with `Bun.password.verify`, checks blocked status, creates Session record. Logs to audit trail.
- Google OAuth: `GET /api/auth/google` → Google → `GET /api/auth/callback/google` — upserts user, creates session
- Session: `GET /api/auth/session` — looks up session by cookie token, returns user (including role & blocked) or 401, auto-deletes expired
- Logout: `POST /api/auth/logout` — deletes session from DB, clears cookie
- Blocked users: login returns 403, existing sessions are invalidated on block, frontend redirects to `/blocked`

## Admin API (SUPER_ADMIN only)

- `GET /api/admin/users` — list all users with role, blocked status, createdAt
- `PUT /api/admin/users/:id/role` — change role to USER or ADMIN (cannot change self or to SUPER_ADMIN)
- `PUT /api/admin/users/:id/block` — block/unblock user (deletes all sessions on block)
- `GET /api/admin/presence` — list online user IDs
- `GET /api/admin/logs/app` — app logs from Redis (filter: level, limit, afterId)
- `GET /api/admin/logs/audit` — audit logs from DB (filter: userId, action, limit)
- `DELETE /api/admin/logs/app` — clear all app logs from Redis
- `DELETE /api/admin/logs/audit` — clear all audit logs from DB
- `GET /api/admin/routes` — all routes metadata (method, path, auth level, category, description) with summary stats
- `GET /api/admin/project-structure` — scans `src/`, `prisma/`, `tests/` — returns files with line counts, exports, imports, categories + directory tree
- `GET /api/admin/env-map` — environment variables with set/unset status, required/optional, default values, consuming files
- `GET /api/admin/test-coverage` — source files + test files mapping, coverage status (covered/partial/uncovered)
- `GET /api/admin/dependencies` — NPM packages from package.json with version, type (runtime/dev), category, importing files
- `GET /api/admin/migrations` — Prisma migration timeline with parsed SQL changes and date info
- `GET /api/admin/sessions` — all active sessions with user info, online status, expiry, role breakdown

## Tickets API

Role-gated ticket tracking. Status machine: `OPEN → IN_PROGRESS → READY_FOR_QC → CLOSED` with `REOPENED` branch. Allowed-transition helper enforces valid moves.

- `GET /api/tickets` — list (QC users see only QC-scope tickets)
- `POST /api/tickets` — create (any authed user reports)
- `GET /api/tickets/:id` — detail with comments + evidence
- `PATCH /api/tickets/:id` — update status/priority/assignee (role-gated)
- `POST /api/tickets/:id/comments` — add comment
- `POST /api/tickets/:id/evidence` — attach evidence (url + kind)

Frontend: `src/frontend/components/TicketsPanel.tsx` — shared between `/dev` and `/dashboard`. Filtered to QC scope when user is QC.

## MCP Server

Local MCP server lets Claude drive the app remotely. `.mcp.json` registers `app-mcp` (runs `scripts/mcp/server.ts`) alongside `playwright`. Requires `MCP_SECRET`; `MCP_SECRET_ADMIN` unlocks write/dev tools.

- Entry: `scripts/mcp/server.ts` + `scripts/mcp/test-client.ts`
- Tool modules (`scripts/mcp/tools/`): `admin`, `code`, `db`, `dev`, `health`, `logs`, `presence`, `project`, `redis`, `tickets`, `shared`
- Ticket tools: `list`, `get`, `claim`, `comment`, `add_evidence`, `ready_for_qc`, `create`, `close`, `reopen`, `update`
- HTTP fallback: `POST /mcp` — readonly with `MCP_SECRET`, full with `MCP_SECRET_ADMIN`

## WebSocket

- `WS /ws/presence` — real-time user presence. Authenticates via session cookie. Tracks connections in-memory (`src/lib/presence.ts`). Broadcasts online user list to admin subscribers on connect/disconnect.

## Logging

Two log systems:

- **App Logs** (`src/lib/applog.ts`) — Redis-backed ring buffer (500 entries). Logs API requests (via `onAfterResponse` hook), errors, auth events. Auto-rotates via `LTRIM`. Can be cleared manually.
- **Audit Logs** (DB `AuditLog` table) — Persistent user activity trail. Actions: `LOGIN`, `LOGOUT`, `LOGIN_FAILED`, `LOGIN_BLOCKED`, `ROLE_CHANGED`, `BLOCKED`, `UNBLOCKED`. Auto-cleanup of records older than `AUDIT_LOG_RETENTION_DAYS` (default 90) runs on startup + every 24h. Can be cleared manually.
- **Pagination** — Dev Console App Logs and User Logs use client-side pagination (25 per page). Avoids rendering hundreds of rows while polling every 5s. Page resets on filter change.

## Role-Based Routing

| Role | Default Route | Can Access |
|------|--------------|------------|
| SUPER_ADMIN | `/dev` | `/dev`, `/dashboard`, `/profile` |
| ADMIN | `/dashboard` | `/dashboard`, `/profile` |
| QC | `/dashboard` | `/dashboard` (QC-scoped tickets only), `/profile` |
| USER | `/profile` | `/profile` |

- `getDefaultRoute(role)` in `src/frontend/hooks/useAuth.ts` — centralized redirect logic
- Blocked users are redirected to `/blocked` from all protected routes
- Tab state persisted in URL search params (`?tab=`) for `/dev` and `/dashboard`

## Frontend

React 19 + Vite 8 (middleware mode in dev). File-based routing with TanStack Router.

- Entry: `src/frontend.tsx` — renders App, removes splash screen, DevInspector in dev
- App: `src/frontend/App.tsx` — MantineProvider (auto color scheme), ModalsProvider (`@mantine/modals`), QueryClientProvider, RouterProvider
- Routes: `src/frontend/routes/`
  - `__root.tsx` — Root layout (renders Outlet only, no floating UI)
  - `index.tsx` — Landing page (theme toggle top-right)
  - `login.tsx` — Login page (email/password + Google OAuth, theme toggle top-right)
  - `dev.tsx` — Dev console with AppShell sidebar: Overview, Users, App Logs, User Logs, Database (React Flow ER diagram), Project (10 sub-views — all React Flow with auto-save), Settings (SUPER_ADMIN only)
  - `dashboard.tsx` — Admin dashboard with AppShell sidebar: Dashboard, Analytics, Orders, Messages, Calendar, Settings (ADMIN+)
  - `profile.tsx` — User profile (all authenticated users, theme toggle in header)
  - `blocked.tsx` — Blocked user page with explanation (theme toggle top-right)
- Components: `src/frontend/components/`
  - `ThemeToggle.tsx` — Shared dark/light mode toggle button (used across all pages)
  - `NotFound.tsx` — 404 page
  - `ErrorPage.tsx` — Error boundary page
- Auth hooks: `src/frontend/hooks/useAuth.ts` — `useSession()`, `useLogin()`, `useLogout()`, `getDefaultRoute()`
- Presence hook: `src/frontend/hooks/usePresence.ts` — WebSocket auto-connect, exposes `onlineUserIds`
- UI: Mantine v8 + `@mantine/modals` (dark/light, auto default from device), react-icons, AppShell layout for dashboard pages
- Sidebar: Collapsible (260px expanded → 60px icon-only minimized with tooltips). State persisted in `localStorage`. Both dev and dashboard use same pattern.
- Logout: Confirm modal via `@mantine/modals` (`modals.openConfirmModal`) on dev, dashboard, and profile pages. Blocked page logs out directly (no confirm).
- Color scheme: `index.html` reads `localStorage` before first paint to prevent flash. Toggle integrated per-page (sidebar footer on AppShell pages, top-right on standalone pages). Persisted by Mantine in `localStorage`.

## Database Schema Visualization

- Dev Console Database tab renders an interactive ER diagram using `@xyflow/react` (React Flow)
- `GET /api/admin/schema` parses `prisma/schema.prisma` into models/fields/relations/enums JSON via `parseSchema()` in `src/app.ts`
- Custom node types: `ModelNode` (table fields with types/attributes) and `EnumNode` (enum values)
- Auto-save to `localStorage`: node positions (`dev:schema:positions`) and viewport/zoom (`dev:schema:viewport`) — debounced 500ms
- On reload, restores last positions and viewport. Falls back to grid layout + fitView if no saved state.

## Project Structure Visualization

- Dev Console Project tab — 10 sub-views switchable via grouped Select dropdown:
  - **Architecture group:**
    - **API Routes**: `GET /api/admin/routes` — all HTTP + WS + frontend routes with method/auth/category badges. Edges show login→redirect flow.
    - **File Structure**: `GET /api/admin/project-structure` — file nodes with import dependency edges. Filter by category. Double-click opens file in editor.
    - **User Flow**: Static — role-based navigation: landing → login → auth → blocked check → role check → destination.
    - **Data Flow**: Static — request lifecycle: client → Elysia → auth → handler → DB/Redis → response. WS + audit flows.
  - **DevOps group:**
    - **Env Variables**: `GET /api/admin/env-map` — env vars with set/unset status, required/optional badges, edges to consuming files.
    - **Test Coverage**: `GET /api/admin/test-coverage` — source files (green/yellow/red coverage) with edges to test files. Filter by coverage status.
    - **Dependencies**: `GET /api/admin/dependencies` — NPM packages by category/type with edges to importing files.
    - **Migrations**: `GET /api/admin/migrations` — horizontal timeline of Prisma migrations with SQL preview and change type badges.
  - **Live group:**
    - **Sessions**: `GET /api/admin/sessions` — active user sessions with online indicator, role mapping. Auto-refresh 10s.
    - **Live Requests**: Real-time API requests via WS broadcast. Hit counters, status color glow, avg response time. Pause/clear controls.
- Each sub-view has independent auto-save (positions + viewport) via `useFlowAutoSave(key)` hook
- All dynamic views have reload buttons. File nodes support double-click to open in editor.
- Request broadcast: `onAfterResponse` hook sends `{ type: 'request', method, path, status, duration }` to admin WS subscribers via `broadcastToAdmins()` in `src/lib/presence.ts`

## Dev Tools

- Click-to-source: `Ctrl+Shift+Cmd+C` toggles inspector. Custom Vite plugin (`inspectorPlugin` in `src/vite.ts`) injects `data-inspector-*` attributes. Reads original file from disk for accurate line numbers.
- HMR: Vite 8 with `@vitejs/plugin-react` v6. `dedupeRefreshPlugin` fixes double React Refresh injection.
- Editor: `REACT_EDITOR` env var. `zed` and `subl` use `file:line:col`, others use `--goto file:line:col`.

## Testing

Tests use `bun:test`. Three levels:

```bash
bun run test              # All tests
bun run test:unit         # tests/unit/ — env, db connection, bcrypt
bun run test:integration  # tests/integration/ — API endpoints via app.handle()
```

- `tests/helpers.ts` — `createTestApp()`, `seedTestUser()`, `createTestSession()`, `cleanupTestData()`
- Integration tests use `createApp().handle(new Request(...))` — no server needed

## APIs

- `Bun.password.hash()` / `Bun.password.verify()` for bcrypt
- `Bun.RedisClient` for Redis (native, no package)
- `Bun.file()` for static file serving in production
- `Bun.which()` / `Bun.spawn()` for editor integration
- `crypto.randomUUID()` for session tokens

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/makuroprojects) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
