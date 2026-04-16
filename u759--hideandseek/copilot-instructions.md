## hideandseek

> This repo is a real-time, location-based multiplayer game with a Spring Boot (WAR) backend and an Expo React Native frontend. It ships an in-browser Admin panel inside the same WAR.


# UBCeek Hide & Seek — AI Coding Assistant Guide

This repo is a real-time, location-based multiplayer game with a Spring Boot (WAR) backend and an Expo React Native frontend. It ships an in-browser Admin panel inside the same WAR.

## Architecture at a glance
- Backend (Java, Spring Boot) — `hideandseek_backend/`
	- In-memory game store (no DB). Core types: `com.hideandseek.model.*`; state access via `com.hideandseek.store.GameStore` and `com.hideandseek.service.GameService`.
	- WebSocket at `/ws` using `GameWebSocketHandler` broadcasting per-game updates; services call `broadcastToGame(gameId, ...)` after mutations.
	- Admin panel served from WAR: `src/main/resources/webapp/{admin-login,admin-dashboard,admin-game}.html` with REST under `/api/admin/**`.
	- Session guard for admin: `com.hideandseek.config.AdminSecurityConfig` checks `ADMIN_AUTH` session attr; login via `/api/admin/auth/login` sets session.
- Frontend (Expo RN, TypeScript) — `frontend/HideAndSeekApp/`
	- Centralized game state over WebSocket: `hooks/useGameState.ts` (see WEBSOCKET_MIGRATION_SUMMARY.md).
	- API and WS config in `config/api.ts` (see `API_BASE_URL` and `getWebsocketUrl()`).

## Critical workflows
- Backend (local dev on Windows PowerShell)
	- Run: `cd hideandseek_backend; .\mvnw.cmd spring-boot:run`
	- Package WAR: `cd hideandseek_backend; .\mvnw.cmd clean package` → `target/hideandseek.war`
	- Admin UI: open `/admin` → login → dashboard (`/admin-dashboard.html`) and live map (`/admin-game.html?gameId=...`).
- Frontend (Expo)
	- `cd frontend/HideAndSeekApp; npm install; npm start`
	- Ensure `API_BASE_URL` points to your backend; WS URL derives from it.

## Project-specific conventions
- No database: game state is ephemeral; server restart clears all games. Always use `GameStore`/`GameService`; do not cache global state elsewhere.
- Real-time pattern: Any state mutation in services must broadcast via `GameWebSocketHandler.broadcastToGame(gameId, payload)` so RN clients and Admin map update instantly.
- Roles and flows: Seekers vs Hiders; completing challenges earns tokens; clues reveal hider locations; hiders share GPS periodically.
- Admin authentication: Password comes from env or `.env` (see `AdminPasswordProvider`) or `application.properties` (`admin.password`). Admin APIs/pages are session-protected by `AdminSecurityConfig`.

## Key integration points
- WebSocket
	- Server: `src/main/java/com/hideandseek/websocket/GameWebSocketHandler.java` at `/ws`.
	- Client: RN uses `getWebsocketUrl()`; Admin map uses native `WebSocket` and sends `{ type: "join", gameId }`.
- REST examples (admin)
	- `GET /api/admin/stats`, `GET /api/admin/games`, `GET /api/admin/games/{id}`
	- `DELETE /api/admin/games/{id}`, `DELETE /api/admin/games/cleanup/{ended|all}`
	- `POST /api/admin/games/{id}/force-end`, `GET /api/admin/games/{id}/live`
- External services
	- Geocoding via OpenCage (see `pom.xml`), AI clue generation via Google GenAI (configure API keys in environment or properties).

## Examples from code
- WS URL derivation: `frontend/HideAndSeekApp/config/api.ts#getWebsocketUrl()` → resolves ws(s)://…/ws from API base.
- Admin guard and redirects: `AdminSecurityConfig` returns 401 JSON for `/api/admin/**` when unauthenticated; HTML requests redirect to `/admin-login.html`.
- Live map source: `admin-game.html` loads `/api/admin/games/{id}/live` for initial snapshot, then listens for `gameUpdate` over `/ws`.

Tip for agents: keep models aligned between backend `com.hideandseek.model` and frontend `frontend/HideAndSeekApp/types.ts`; if you add fields, update both and ensure broadcasts include the new data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/u759) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
