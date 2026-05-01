## antigravity-ide-mobile

> **Generated:** 2026-03-10

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-10
**Commit:** 9a1efd5
**Branch:** main

## OVERVIEW
Mobile dashboard for Antigravity IDE (Live chat view and remote control). Monorepo with a Node.js/Express TypeScript backend (`server/`) and Preact/Vite frontend (`client/`), bridging interactions via REST, WebSockets, and Chrome DevTools Protocol (CDP).

## STRUCTURE
```
.
├── client/   # Preact frontend (v2.0)
├── server/   # Node.js Express backend (v2.0)
├── src/      # Legacy v1.x JavaScript backend implementation (do not modify)
├── public/   # Legacy v1.x frontend files (do not modify)
└── data/     # Local JSON persistence storage
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Frontend UI components | `client/src/components/` | Preact components |
| API / WS integration | `client/src/hooks/` | See `useApi.ts` and `useWebSocket.ts` |
| Express routing | `server/src/routes/` | Route factories handling API endpoints |
| Server logic / services | `server/src/services/` | Business logic (CDP, AI Supervisor, Tunnel) |
| Local persistence | `server/src/config.ts` | JSON file-based storage |
| Legacy Reference | `src/` & `public/` | Ported implementations for reference only |

## CONVENTIONS
- **V2.0 Monorepo**: All new development happens in `client/` and `server/`.
- **Database**: The project eschews traditional ORMs/DBs. It uses flat JSON files in `data/` for configuration, messages, and analytics, plus in-memory state for active sessions.
- **Dependency Injection**: Server routes use a factory pattern (`createXRoutes(deps)`) to inject services and config, avoiding global state.
- **Frontend State**: Preact Context (`AppContext`) handles global state; `useRef` bridges high-frequency WebSocket events to avoid re-renders.
- **Custom Routing**: The frontend uses custom hash-based routing (`app.tsx`) rather than a heavy library like React Router.

## ANTI-PATTERNS (THIS PROJECT)
- DO NOT edit files in the root `src/` or `public/` directories for new features. They are legacy implementations.
- DO NOT introduce heavy frontend libraries (e.g., Redux, React Router). The architecture relies on lightweight custom implementations (Context, hash-router, native fetch wrapper).
- DO NOT add a SQL/NoSQL database (like Postgres or MongoDB) without explicit architectural changes; rely on JSON persistence in `data/`.
- DO NOT handle API calls directly via `fetch` in the frontend; use the custom `authFetch` wrapper in `useApi.ts`.

## COMMANDS
```bash
# Development (Start both client and server)
npm run dev
# Build (Build both)
npm run build
# Start production server
npm run start
```

## NOTES
- The mobile client controls the desktop IDE remotely via the Chrome DevTools Protocol (CDP).
- Live chat synchronization uses `morphdom` to efficiently patch the DOM received from the IDE.

---
> Source: [mrkungfudn/antigravity-ide-mobile](https://github.com/mrkungfudn/antigravity-ide-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
