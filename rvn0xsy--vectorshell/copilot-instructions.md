## vectorshell

> This file provides guidance when working in this repository.

# CLAUDE.md

This file provides guidance when working in this repository.

## Development commands

### Makefile
- `make build` — Build server and client binaries
- `make build-server` — Build only the server
- `make build-client` — Build only the client
- `make run-server` — Run the server with `./config.toml`
- `make run-client` — Run the client with `./config.toml`
- `make run-repl` — Run the local REPL
- `make test` — Run all Go tests
- `make web-dev` — Start the dashboard Vite dev server
- `make web-build` — Build the dashboard for production
- `make docker-build` — Build the Docker image

### Go commands
- `go run ./cmd/server -config ./config.toml` — Run the server
- `go run ./cmd/client -config ./config.toml` — Run the client
- `go run ./cmd/repl -config ./config.toml` — Run the REPL
- `go test ./...` — Run all Go tests
- `go build ./cmd/server` — Build the server
- `go build ./cmd/client` — Build the client

### Web app
- `npm install --prefix dashboard` — Install dependencies
- `npm run --prefix dashboard dev` — Start Vite dev server
- `npm run --prefix dashboard build` — Build production assets
- `npm run --prefix dashboard lint` — Lint the frontend

## Configuration and runtime shape
- Main runtime config: `config.toml` at the repository root.
- Example config: `config.example.toml`.
- Agent instruction file: `SOUL.md` at the repository root, referenced by `agent.soul_path`.
- The server serves the frontend from `dashboard/dist` at `ui_path`.
- Skill documents are loaded from `skill/` when enabled.
- Persistent SQLite database: `data/vectorshell-go.db`.
- Client artifacts are produced under `build/clients/`.

## High-level architecture

### Workspace structure
- `cmd/server`: server entrypoint.
- `cmd/client`: client entrypoint.
- `cmd/repl`: REPL entrypoint.
- `internal/api`: HTTP API, WebSocket registration, SSE, MCP, static UI serving.
- `internal/agent`: Eino-based agent service and remote tool wrapping.
- `internal/client`: reverse-connecting client runtime and local tool executor.
- `internal/session`: live session registry and tool dispatch.
- `internal/store`: SQLite persistence for conversations and artifacts.
- `internal/events`: in-memory broadcast bus for SSE streams.
- `internal/protocol`: shared WebSocket message schema.
- `dashboard`: React/Vite frontend.
- `skill`: skill markdown content consumed by the agent.

### Server architecture
- `cmd/server/main.go`: loads config, creates session manager, agent service, SQLite store, then starts the HTTP server.
- `internal/api/server.go`: main server boundary for REST, SSE, WebSocket, artifact handling, MCP, and static UI routing.
- `internal/session/manager.go`: stores active connections, pending tool calls, and file transfer helpers.
- `internal/store/sqlite.go`: persistent conversation and artifact metadata store.
- `internal/agent/service.go`: constructs the Eino agent, registers remote tools, and emits conversation events.

### Client architecture
- `cmd/client/main.go`: loads config and starts the reverse WebSocket client.
- `internal/client/websocket.go`: registration, heartbeats, server message handling, and tool execution dispatch.
- `internal/client/executor.go`: local implementations for exec, read_file, write_file, upload_file, and download_file.

### Frontend architecture
- `dashboard/src/App.tsx`: main UI component.
- The frontend talks only to the HTTP API and SSE endpoints.
- The server mounts built assets at `/ui`.

### Notes
- The previous Rust workspace is no longer the primary implementation.
- `dev-go/eino-examples` is reference material and not part of the main runtime.

---
> Source: [Rvn0xsy/vectorshell](https://github.com/Rvn0xsy/vectorshell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
