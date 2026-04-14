## tierlist-maker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start              # Dev server: Vite frontend + Wrangler API concurrently
npm run dev            # Vite only (port 5173)
npm run dev:api        # Wrangler Pages Functions only (port 8788, proxied from Vite)
npm run build          # Production build (vite build)
npm run lint           # ESLint
npm run deploy         # Build + deploy to Cloudflare Pages (manual, not git-triggered)
```

Local dev requires `.dev.vars` for server-side secrets (DATABASE_URL, JWT_SECRET, DISCORD_BOT_TOKEN, ANTHROPIC_API_KEY, etc.) and `.env` for frontend vars (VITE_DISCORD_CLIENT_ID, VITE_DISCORD_REDIRECT_URI). Vite proxies `/api/*` to Wrangler at port 8788.

## Architecture

**Frontend**: React 19 + Vite 7 + Tailwind CSS 4 + React Router 7 (SPA)
**Backend**: Cloudflare Pages Functions at `functions/api/*.js` — one file per endpoint, all wrapped with `adapt()` from `functions/lib/adapter.js`
**Database**: PostgreSQL on Neon via `@neondatabase/serverless` tagged template literals (injection-safe by design)
**Auth**: Discord OAuth → JWT (jose library, async sign/verify) → RBAC permissions
**Storage**: Cloudflare R2 for images (team icons, codex images), binding: `TEAM_ICONS`

### Request Flow

1. Frontend calls `apiCall(endpoint, params)` or `apiPost(endpoint, params, body)` from `src/services/database.js` — auto-injects auth token + impersonation header
2. Vite proxies to Wrangler → Cloudflare Pages Function at `functions/api/{endpoint}.js`
3. `adapt(handler)` converts CF Request → event object with `{ request, env, httpMethod, body, queryStringParameters, waitUntil }`
4. Handler runs auth check (`requireAuth` or `requirePermission` from `functions/lib/auth.js`), then business logic with `neon()` tagged templates from `functions/lib/db.js`
5. Returns `{ statusCode, headers, body }` — admin endpoints use `adminHeaders(event)` (ALLOWED_ORIGIN CORS), public use `headers` (wildcard)

### Key Patterns

- **Division-scoped routing**: `/:leagueSlug/:divisionSlug/*` — `DivisionContext` provides league/division/season/teams/players to all child routes
- **RBAC**: `requirePermission(event, 'permission_key', leagueId)` on every admin endpoint. Frontend uses `hasPermission(key, leagueId?)` from AuthContext
- **Transactions**: `await transaction(event, async (sql) => { ... })` — uses WebSocket Client (not HTTP) for persistent connections
- **Module-level process.env reads are forbidden** — CF env bindings aren't available until request time; always read inside handler functions
- **Fire-and-forget**: CF kills execution after Response returns — use `event.waitUntil(promise)` for background work (audit logs, forge updates, prediction resolution)
- **Large pages split into subdirectories**: parent keeps state + orchestration, children are presentational via props (e.g., `AdminDashboard.jsx` imports from `dashboard/`, `RosterManager.jsx` from `roster/`, `Predictions.jsx` from `predictions/`)

### Context Providers

| Context | Scope | Key State |
|---------|-------|-----------|
| `AuthContext` | Global | user, token, permissions, `hasPermission()`, login/logout, impersonation |
| `DivisionContext` | `/:leagueSlug/:divisionSlug/*` | league, division, season, teams, players |
| `PassionContext` | Global | balance, rank, streak, daily claim, challenge notifications |
| `SidebarContext` | Global | sidebar open/close toggle |

### Layout Hierarchy

`AppLayout` (auth, sidebar, global modals) → `AdminLayout` | `CodexLayout` | `DivisionLayout` | standalone pages

## Hard Rules

- Never call Passion "coins" or "Passion Coins" — always just "Passion"
- Predictions page/links must never appear on public (non-admin) pages
- KDA formula is intentionally `(K + A/2) / D` (not standard MOBA formula)
- `VITE_` prefix exposes env vars to frontend bundle — never use for secrets
- `.dev.vars` replaces `.env` for local server-side development with Wrangler

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EricdeZ)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EricdeZ)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
