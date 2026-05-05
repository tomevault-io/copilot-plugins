## hyscale

> Web-based management and launcher for Hytale game servers, accessible via Tailscale private network.

# hyscale - Hytale Server Manager

Web-based management and launcher for Hytale game servers, accessible via Tailscale private network.

## Stack

- **Backend:** Go 1.25 (Tailscale tsnet, Gorilla Mux/WebSocket, OAuth2)
- **Frontend:** Vanilla JS with xterm.js (terminal) and Monaco Editor (config editing)
- **Networking:** Tailscale private mesh network, HTTPS on 443, UDP proxy on 5520

## Project Structure

```
cmd/hyscale/main.go       # Application entrypoint & bootstrap
cmd/hyscale/web/          # Embedded web UI assets
internal/api/              # HTTP handlers & WebSocket (handlers.go, websocket.go)
internal/server/           # Process lifecycle management (process.go)
internal/auth/             # OAuth2 authentication
internal/downloader/       # Server binary download/update
internal/hytaleapi/        # Hytale API client
internal/config/           # Application config management
internal/backup/           # Backup/restore functionality
internal/logs/             # Log streaming
server/                    # Runtime data (config, worlds, mods, logs)
```

## Build & Run

```bash
# Build
go build -o hyscale.exe ./cmd/hyscale

# Run (requires Tailscale auth on first run)
./hyscale.exe
```

Web assets are embedded into the binary via `//go:embed`.

## Configuration

- `hyscale.json` - Main app config (Tailscale node name, ports, JVM settings)
- `server/config.json` - Hytale server configuration
- `.hytale-credentials.json` - OAuth tokens (git-ignored, runtime generated)

## Architecture

1. Bootstrap initializes Tailscale connection and authenticates
2. HTTPS server starts on Tailscale network
3. ProcessManager controls Hytale server subprocess (start/stop/commands)
4. API handlers expose REST endpoints for all management operations
5. SSE provides real-time console streaming
6. UDP proxy forwards game traffic through Tailscale

## Key Patterns

- Tailscale tsnet for private networking and authentication
- Process management with stdio piping for console I/O
- OAuth2 device auth flow for Hytale API access
- Server-Sent Events for log streaming
- JSON config persistence with schema validation

## Local Commands

* When invoking commands on windows (bash tool), use forward slashes, not backslashes.
* You do not need to cd to run comamnds, e.g. just `go build ./cmd/hyscale` from the project root.
* Windows does not have `dir /b` anymore, and you must never do `2>nul` that creates hard to delete files.

---
> Source: [tailscale/hyscale](https://github.com/tailscale/hyscale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
