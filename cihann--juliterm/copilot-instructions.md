## juliterm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Juli

Juli is a web-based tmux client. It runs a Go server that exposes local tmux sessions to browser clients via WebSocket + xterm.js. Each browser tab gets its own PTY running `tmux attach-session`, so it behaves exactly like a native terminal.

## Build & Run

```bash
# Full production build (Nuxt SSG → embedded in Go binary)
make build          # outputs ./juli

# Development (two terminals)
go run ./cmd/juli --token dev --port 8080
cd web && npm run dev

# Run tests
go test ./...

# Cross-compile releases
make release
```

Authentication requires `--token <secret>` or `JULI_TOKEN` env var. The host must have `tmux` installed.

## Architecture

### Go backend (`cmd/juli`, `internal/`)

**Entry point**: `cmd/juli/main.go` — parses flags, creates `tmux.Manager`, starts `server.Server`.

**`internal/tmux`** — Manages tmux interaction:
- `Manager` — discovers sessions via `tmux list-sessions`, spawns a `ControlClient` per session, maintains a unified `SessionTree`. A dedicated watcher `ControlClient` listens for `%sessions-changed` to auto-sync.
- `ControlClient` (`control.go`) — runs `tmux -C attach-session` (control mode) to receive real-time events (`%window-add`, `%window-close`, etc.).
- `SessionTree` (`sessions.go`) — thread-safe tree of Session → Window → Pane, serializable to JSON for the frontend.

**`internal/pty`** — Per-connection PTY. Each WebSocket client gets a `pty.Session` that runs `tmux attach-session -t <name>` in a real PTY. Raw bytes flow between the PTY fd and the WebSocket.

**`internal/server`** — HTTP/WebSocket server:
- `server.go` — route setup, SPA file serving, TLS support.
- `ws.go` — WebSocket lifecycle: upgrade → send session tree → read loop dispatching `MsgInput`/`MsgResize`/`MsgSessionTree` → spawn PTY on session select → bridge PTY↔WS.
- `fanout.go` — broadcasts session tree changes to all connected clients.
- `auth.go` — token-based login via `POST /api/login`, session cookie validation.
- `api.go` — `GET /api/sessions` returns current session tree JSON.

**`internal/protocol`** — Binary WebSocket protocol. Every frame: `[1-byte type | payload]`. Types: `0x01` Output, `0x02` Input, `0x03` Resize, `0x04` SessionTree, `0x07` Error.

### Nuxt frontend (`web/`)

Nuxt 4 + Vue 3 + xterm.js SPA. In production, statically generated (`nuxi generate`) and embedded into the Go binary via `embed.FS`.

- `composables/useTerminalSocket.ts` — WebSocket connection, binary protocol encode/decode, xterm.js integration.
- `composables/useSessionTree.ts` — reactive session tree state from server broadcasts.
- `components/TerminalView.client.vue` — client-only xterm.js terminal renderer.
- `components/SessionTree.vue` — sidebar showing sessions/windows/panes.
- `middleware/auth.ts` — redirects to `/login` if no session cookie.

### Embedding

`cmd/juli/embed.go` uses `//go:embed` to bundle the Nuxt static output (`cmd/juli/web/`) into the Go binary. The `Makefile` copies `web/.output/public` → `cmd/juli/web` before `go build`.

## WebSocket Protocol Flow

1. Client connects to `/ws` with session cookie
2. Server sends `MsgSessionTree` (0x04) with full tree JSON
3. Client sends `MsgSessionTree` (0x04) with `{"sessionName": "..."}` to attach
4. Server spawns PTY → streams `MsgOutput` (0x01) to client
5. Client sends `MsgInput` (0x02) for keystrokes, `MsgResize` (0x03) for terminal resize

---
> Source: [cihann/juliterm](https://github.com/cihann/juliterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
