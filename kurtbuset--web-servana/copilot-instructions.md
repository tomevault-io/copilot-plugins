## web-servana

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About

`web_servana` is the React 19 / Vite 6 agent dashboard for the Servana customer service platform. It runs on port 3000 in Docker or port 5173 in local dev.

## Commands

```bash
npm install
npm run dev        # Vite dev server, port 5173
npm run build      # production build → dist/
npm run lint       # ESLint
npm run preview    # preview production build
```

No test runner is configured in this repo.

## Environment Variables

Required in a `.env` file at repo root:
- `VITE_BACKEND_URL` — backend API base URL (e.g. `http://localhost:5000`)
- `VITE_SOCKET_URL` — Socket.IO server URL (falls back to `VITE_BACKEND_URL`)
- `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY` — injected as Docker build args from root `docker-compose.yml`

## Architecture

### Entry Points
- `src/App.jsx` — mounts all Context providers in order, then `<AppNavigation />`
- `src/AppNavigation.jsx` — React Router setup; all routes are lazy-loaded except Login

### Provider Stack (outermost → innermost)
```
ThemeProvider → UserProvider → RolePreviewProvider → AgentStatusProvider → UnsavedChangesProvider → DepartmentPanelProvider
```

### Auth & Permissions
- `UserProvider` (`src/context/UserContext.jsx`) is the central auth source. It fetches the user profile on mount and exposes `hasPermission(permissionKey)` — permission keys follow the `priv_can_*` naming scheme defined by the backend.
- `ProtectedRoute` checks `GET /auth/me` on every navigation; `PermissionRoute` wraps it to gate by specific `priv_can_*` keys.
- `tokenService` (`src/services/token.service.js`) handles automatic JWT refresh every 12 minutes (tokens expire at 15 min) — started inside `UserProvider`.
- Auth uses **HTTP-only cookies** (no localStorage tokens). The `api` Axios instance is configured with `withCredentials: true`.

### API Layer
- `src/api.js` — single Axios instance; unwraps `{ data: ... }` envelopes automatically so callers see `response.data` directly.
- `src/services/*.service.js` — one file per domain (auth, agent, chat, department, role, macro, queue, dashboard, analytics, profile, autoReply). All are static class methods calling `api`.

### Real-time (Socket.IO)
- `src/socket/index.js` — creates the singleton socket with `autoConnect: false`; auth reads the `access_token` cookie.
- `UserContext` connects/disconnects the socket based on `userData?.sys_user_id`.
- Domain-specific socket logic lives in `src/socket/chat.js`, `agent.js`, `typing.js`, `auth.js`, `connection.js`.
- React hooks like `useChatSocket`, `useSocketConnection`, `useTyping`, `useAgentStatusSocket` wrap the socket events for component use.

### Views
Each view lives in `src/views/<name>/` with a top-level `<ScreenName>.jsx` and a `components/` subfolder. Views map 1-to-1 with routes in `AppNavigation.jsx`:
- `/Dashboard`, `/Chats`, `/ResolvedChats`, `/department`, `/manage-agents`, `/roles`, `/change-role`, `/auto-replies`, `/agents` (macros), `/manage-admin`, `/analytics`, `/profile`

### Hooks
`src/hooks/` contains domain-specific hooks that call services and manage local state. Naming convention: `use<Domain>.js`. Socket-specific hooks are named `use<Domain>Socket.js`.

### Constants
`src/constants/config.js` exports `ROLE_IDS`, `CHAT_STATUS`, `STORAGE_KEYS`, feature flags (`FEATURES`), UI config, and validation rules. Import via barrel: `import { ROLE_IDS } from '../constants'`.

---
> Source: [kurtbuset/web_servana](https://github.com/kurtbuset/web_servana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
