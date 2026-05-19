## cli

> Codewire is a persistent process server for AI coding agents. Single Go binary (`cw`) acts as both node and CLI client. Manages PTY sessions that survive disconnects — launch AI agents, detach, reconnect later.

# Claude Code Context

## Project Overview

Codewire is a persistent process server for AI coding agents. Single Go binary (`cw`) acts as both node and CLI client. Manages PTY sessions that survive disconnects — launch AI agents, detach, reconnect later.

Two tiers: **Standalone** (default, zero config, works like tmux) and **Relay mode** (opt-in remote access via WireGuard, network discovery, shared KV storage).

## Tech Stack

Go 1.23+, cobra, creack/pty, nhooyr.io/websocket, coder/wgtunnel, modernc.org/sqlite, BurntSushi/toml, golang.org/x/term

## Project Structure

```
cmd/cw/main.go              # CLI entry (cobra)
internal/
  auth/auth.go              # Token generation/validation
  config/config.go          # TOML config + env overrides
  protocol/
    protocol.go             # Frame wire format [type:u8][len:u32 BE][payload]
    messages.go             # Request/Response JSON (PascalCase type discriminator)
  connection/
    connection.go           # FrameReader/FrameWriter interfaces
    unix.go                 # Unix socket transport
    websocket.go            # WebSocket transport (Text=Control, Binary=Data)
  session/
    session.go              # SessionManager, Broadcaster, StatusWatcher, PTY lifecycle
    events.go               # Event types, SubscriptionManager, EventLog (JSONL)
  node/
    node.go                 # Node: Unix listener, WS server, PID file, signals
    handler.go              # Client dispatch, attach/watch/logs/subscribe/wait handlers
  client/
    client.go               # Target (local/remote/relay), Connect, requestResponse
    commands.go             # All CLI command implementations (merged network messaging)
  terminal/
    rawmode.go              # RawModeGuard (golang.org/x/term)
    size.go                 # Terminal size, SIGWINCH
    detach.go               # DetachDetector state machine (Ctrl+B d)
  statusbar/statusbar.go    # Status bar rendering
  tunnel/
    keys.go                 # WireGuard key management (LoadOrGenerateKey)
    tunnel.go               # Node WireGuard tunnel (NodeTunnel)
    relay.go                # Relay server (WireGuard + HTTP API + device auth)
    setup.go                # Device authorization flow (cw setup)
  store/
    store.go                # Store interface (KV, nodes, device codes)
    sqlite.go               # SQLite implementation (relay-only)
  mcp/server.go             # MCP JSON-RPC over stdio (14 tools)
tests/
  integration_test.go       # E2E tests (core functionality)
  events_test.go            # Event system tests (tags, subscribe, wait)
```

## Development

```bash
make build          # Build ./cw binary
make test           # Unit tests (internal packages)
make lint           # go vet
make test-manual    # CLI smoke test
make install        # Build + install to /usr/local/bin

# All tests including integration
go test ./internal/... ./tests/... -timeout 120s -count=1
```

## CI/CD

- **Gitea** (`.gitea/workflows/ci.yaml`) — builds and deploys the docs website only. Triggers on push to main and version tags.
- **GitHub** (`.github/workflows/ci.yml`) — runs `go test`, `go vet`, and `go build` on ubuntu + macOS. Triggers on push to main and PRs.
- **GitHub Release** (`.github/workflows/release.yml`) — triggered by `v*` tags. Cross-compiles for linux/amd64, linux/arm64, darwin/amd64, darwin/arm64 with `CGO_ENABLED=0`. Creates GitHub release with GPG-signed checksums and updates the Homebrew formula.

### Release Process

```bash
git tag v0.X.Y
git push origin v0.X.Y
```

GitHub Actions builds binaries, creates the release, and updates `Formula/codewire.rb`.

## Git Remotes

`origin` has two push URLs (GitHub + Gitea). CI auto-commits brew formula updates to `main` on GitHub, which can cause GitHub to be ahead of local. Always `git pull --rebase` before pushing to avoid rejected pushes. If Gitea diverges, force push to Gitea (`git push gitea main --force`) — GitHub is the primary remote.

## Key Architecture

- **Wire protocol**: `[type:u8][length:u32 BE][payload]` — type 0x00 = Control (JSON), 0x01 = Data (raw bytes)
- **JSON messages**: flat struct with `"type"` discriminator in PascalCase (e.g. `{"type":"Launch","command":["bash"]}`)
- **Socket**: `~/.codewire/codewire.sock` (Unix domain socket)
- **Session lifecycle**: 3 goroutines per session (PTY reader, input writer, process waiter)
- **Events**: Sessions emit typed events (session.created, session.status, session.output_summary). Events stored in `events.jsonl` per session. SubscriptionManager for pub/sub fan-out.
- **Broadcaster**: fan-out to multiple attached clients, non-blocking send with drop for slow consumers
- **Relay**: WireGuard tunnel via `coder/wgtunnel`. Relay runs embedded SQLite for node registry + shared KV store. Nodes connect via userspace WireGuard (no root needed).
- **Merged CLI**: No separate network namespace. `node:session` syntax everywhere. `cw nodes`, `cw subscribe`, `cw wait` are top-level commands.

---
> Source: [codewiresh/cli](https://github.com/codewiresh/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
