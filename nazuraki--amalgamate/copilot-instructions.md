## amalgamate

> **At the start of every session, read `CONTEXT.md` in this directory.** It contains the complete code reference — file maps, all exported types/functions, DB schema, API surfaces, and data structures. This eliminates the need to re-read source files.

# Amalgamate — Claude Instructions

## First Step: Read CONTEXT.md

**At the start of every session, read `CONTEXT.md` in this directory.** It contains the complete code reference — file maps, all exported types/functions, DB schema, API surfaces, and data structures. This eliminates the need to re-read source files.

## Project Overview

Amalgamate is an Electron desktop app: a Go backend (`backend/`) handles torrent downloading via anacrolix/torrent + SQLite, a Svelte frontend (`frontend/`) is the UI, and Electron (`electron/`) glues them together. The binary is compiled to `electron/resources/amalgamate-backend.exe`.

- **Build backend:** `cd backend && go build -o ../electron/resources/amalgamate-backend.exe ./cmd/ntl-backend`
- **Dev:** `npm run dev` from monorepo root (starts Vite on 5173, then Electron)
- **Go binary:** `C:\Program Files\Go\bin\go.exe`; use `export PATH="$PATH:/c/Program Files/Go/bin"` in Bash

## Canonical Source Locations

The **only** canonical backend source is under `backend/`. Root-level `*.go` files are stale scaffolding — ignore them.

| Layer | Location |
|---|---|
| Go module root | `backend/` |
| API layer | `backend/internal/api/` |
| Config | `backend/internal/config/` |
| Database | `backend/internal/db/` |
| Torrent engine | `backend/internal/torrent/` |
| Frontend | `frontend/src/` |
| Electron | `electron/src/` |

## Coding Conventions

### Go (Backend)
- Package names: `api`, `config`, `db`, `torrent` — match directory names exactly
- IDs: UUIDs via `uuid.New().String()` (see `torrent/id.go`)
- DB queries: raw `database/sql`, no ORM; always scan into typed structs
- JSON responses: use `writeJSON(w, status, v)` and `writeError(w, status, msg)` helpers
- DB rows: Nullable fields use pointer types (`*string`, `*int64`)
- Error handling: Return errors up; handlers convert to HTTP status codes
- Torrent status values: `downloading | seeding | paused | stopped | error`
- Link strategies: `none | hardlink | symlink`
- Auth: Check `cfg.APIKey == ""` for local-mode bypass before key comparison

### Svelte (Frontend)
- State lives in `src/lib/nodes.js` Svelte stores — do not duplicate state in components
- API calls go through `clientForNode(nodeId)` — never raw `fetch` with hard-coded URLs
- Events: use Svelte `createEventDispatcher` for child→parent communication
- Byte formatting: `fmtBytes(b)` helper (duplicated in TorrentRow + TorrentDetail — keep in sync)

### Electron
- IPC handlers are in `electron/src/main.js`; exposed via `window.ntl.*` in `preload.js`
- Never add node integration to BrowserWindow — use context bridge only
- Backend port is discovered from stdout `LISTENING:<port>` pattern

## Key Patterns

- **New DB ID:** `newID()` from `torrent/id.go` (UUID v4)
- **DB open:** WAL mode + foreign keys enforced; always `MaxOpenConns=1`
- **Torrent insert:** `insertOrSkip` — upsert by `(node_id, info_hash)` unique index
- **Progress updates:** `progressLoop` emits every 1s; `watchLoop` scans folders every 5s
- **WebSocket broadcast:** call `s.engine.SetBroadcaster(hub.Broadcast)` once in `NewServer`; engine calls `e.broadcast(evt)` internally
- **Pausing:** cancel all pieces via `t.CancelPieces(0, t.NumPieces())`; resuming calls `t.DownloadAll()`
- **File priority:** deselected files get `PiecePriorityNone`; selected get `PiecePriorityNormal`

## Key Gotchas

- SQLite driver is `modernc.org/sqlite` (pure Go, no CGO) — import with blank `_ "modernc.org/sqlite"`
- Only `MaxOpenConns=1` works with SQLite WAL — do not raise this
- The Electron `preload.js` must use CommonJS (`require`, not `import`) due to Electron preload constraints
- Vite base must be `'./'` (relative) for the frontend to work from `file://` protocol in packaged Electron
- `getBackendBinary()` uses `process.resourcesPath` in packaged mode and `../resources/` in dev — do not add new binary lookup paths without updating both
- When adding a new IPC channel, add it to both `ipcMain.handle()` in `electron/src/main.js` AND expose it in `electron/src/preload.js` via `contextBridge`
- New API endpoints must be registered in `buildRouter()` in `server.go` — handlers alone are not wired up automatically
- Watch folder scan runs `AddTorrentFile`, which calls `insertOrSkip` — idempotent by design

---
> Source: [nazuraki/amalgamate](https://github.com/nazuraki/amalgamate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
