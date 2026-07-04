## terminar

> **NEVER use mock PTY, mock data, or any mock/stub options unless the user explicitly asks for it.** Always use `pnpm dev` (real PTY). Always use `cargo run -- --no-auth`. If a real dependency is unavailable, fail explicitly rather than silently using fakes.

# CLAUDE.md - terminar

## No Mocking Policy (MANDATORY)

**NEVER use mock PTY, mock data, or any mock/stub options unless the user explicitly asks for it.** Always use `pnpm dev` (real PTY). Always use `cargo run -- --no-auth`. If a real dependency is unavailable, fail explicitly rather than silently using fakes.

## Overview

Persistent terminal sessions that survive crashes and restarts. A Rust server manages PTY sessions on a Unix socket, with an Electron tray app as the distributed frontend. Think of it as a built-in tmux.

**Full architecture reference:** [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)

## Directory Structure

```
terminar/
├── server/                  # Rust backend (terminar-server binary)
│   └── src/
│       ├── main.rs          # Server entry point
│       ├── lib.rs           # Server core: routing, message dispatch
│       ├── config.rs        # CLI args (clap)
│       ├── handlers/        # Message handlers (session, I/O, workspace)
│       ├── messages.rs      # Wire protocol types (Rust side)
│       ├── connection.rs    # Client connection tracking
│       ├── settings.rs      # Per-session settings
│       ├── workspace.rs     # Workspace persistence
│       ├── audit.rs         # Event logging
│       ├── constants.rs     # Defaults (ports, timeouts, limits)
│       ├── error.rs         # Error types
│       └── logging.rs       # Tracing configuration
├── web/                     # Web frontend (Svelte 5 + xterm.js + Vite) — dev-only, not shipped
│   └── src/
│       ├── App.svelte       # Root component
│       ├── components/      # ~20 components: Terminal, Pane, WorkspaceView, Sidebar, etc.
│       └── lib/             # ~50 modules: stores, managers, keybindings, themes
├── tray/                    # System tray app (Electron + Svelte 5) — shipped desktop app
│   └── src/
│       ├── main/            # Electron main process (tray, health, server, config, IPC)
│       ├── preload/         # contextBridge API (trayAPI)
│       └── renderer/        # Svelte: Settings window, embedded terminal
├── extension/               # VS Code extension (TypeScript)
│   └── src/                 # ServerController, SessionManager, NetSocketAdapter
├── packages/
│   └── shell-protocol/      # Shared TypeScript protocol (@narai/terminar-protocol)
│       └── src/             # Zod schemas, ShellClient, BaseWebSocketManager
├── npm/                     # npm distribution packages
│   ├── terminar/            # Main CLI package (bin/, lib/, app/)
│   ├── darwin-arm64/        # @narai/terminar-darwin-arm64 (server binary)
│   ├── darwin-x64/          # @narai/terminar-darwin-x64
│   ├── linux-x64/           # @narai/terminar-linux-x64
│   ├── linux-arm64/         # @narai/terminar-linux-arm64
│   └── win32-x64/           # @narai/terminar-win32-x64 (deferred)
├── docs/                    # API docs, architecture plans
│   ├── ARCHITECTURE.md      # Detailed architecture reference (READ THIS)
│   ├── API.md               # Protocol & REST API reference
│   └── plans/               # Design documents for major features
└── package.json             # Root workspace (pnpm monorepo)
```

## Architecture

### Local-Only Mode

```
Electron Tray (tray/) ──WebSocket──► terminar-server ◄──Unix Socket──► VS Code Extension
       │                                    │
  Embedded terminal                    PTY Sessions (portable-pty)
  Settings window
  Health polling
```

The server listens on a Unix socket and localhost HTTP. The Electron tray app is the shipped desktop frontend. The web frontend (`web/`) is used only during development.

### Components

- **terminar-server** (`server/`): Local PTY server. Unix socket + localhost HTTP/WebSocket. Token auth via `~/.terminar/token`.
- **Electron Tray** (`tray/`): Shipped desktop app. System tray icon, embedded terminal, settings, health polling. Bundles the server binary.
- **Web Frontend** (`web/`): Svelte 5 + xterm.js. Dev-only — not shipped in the distributed app.
- **VS Code Extension** (`extension/`): Connects via Unix socket with length-prefixed framing.
- **Protocol Package** (`packages/shell-protocol/`): Zod schemas, ShellClient, BaseWebSocketManager. Shared by web + extension.

## Build & Test Commands

### Root-level shortcuts (preferred)

```bash
pnpm dev                     # Start server (detached) + tray (USE THIS BY DEFAULT)
pnpm dev:web                 # Start server (detached) + web (for web frontend development)
pnpm dev:all                 # Start server (detached) + web + tray (everything)
pnpm server:status           # Check if the detached dev server is running
pnpm server:stop             # Stop the detached dev server
pnpm server:restart          # Rebuild and restart the detached dev server
pnpm server:logs             # Tail the detached dev server's log file
pnpm build                   # Build server (release) + tray
pnpm test                    # Test server
pnpm test:server             # Test server only
pnpm test:web                # Test web only
pnpm test:extension          # Test extension only
pnpm tray:dev                # Run tray app only (Electron dev; assumes server already running)
pnpm tray:test               # Test tray app
```

> **Server is detached from the UI.** `pnpm dev` starts the Rust server as a
> detached background process (shared PID file at `~/.terminar/server.pid`).
> Closing the tray, crashing Electron, or Ctrl+C on the `pnpm dev` terminal
> all leave the server running — reopen the UI and it reattaches to existing
> sessions. To actually stop the server, use `pnpm server:stop`.
> The legacy `pnpm server:dev` script is still available for server-focused
> iteration (runs attached; Ctrl+C kills it).

### Rust Server

```bash
cd server
cargo build                  # Debug build
cargo build --release        # Release build (outputs terminar-server)
cargo test                   # Run tests
cargo clippy                 # Lint
cargo run -- --no-auth       # Dev mode (USE THIS BY DEFAULT)
```

### Web Frontend (dev-only)

```bash
cd web
pnpm dev                     # Vite dev server (localhost:3001)
pnpm build                   # Production build
pnpm test -- --run           # Vitest (single run)
pnpm test:watch              # Vitest (watch mode)
```

### System Tray (Electron)

```bash
cd tray
pnpm dev                     # Run in dev mode (Electron + Vite)
pnpm test -- --run           # Vitest unit tests
npx playwright test          # Playwright E2E tests
pnpm build                   # Build distributable (electron-builder)
```

### Protocol Package

```bash
cd packages/shell-protocol
pnpm build                   # TypeScript → dist/ (MUST run before extension tests)
pnpm test -- --run           # Vitest
pnpm typecheck               # tsc --noEmit (has dead_code check - no unused imports!)
```

### VS Code Extension

```bash
cd extension
pnpm compile                 # TypeScript compilation
pnpm test                    # Mocha tests (requires protocol dist/)
```

### Build Order (when building from scratch)

1. `packages/shell-protocol` → `pnpm build`
2. `server` → `cargo build` (independent of TypeScript)
3. `extension` → `pnpm compile` (depends on shell-protocol dist/)
4. `web` → uses relative imports via `shared-protocol.ts` barrel
5. `tray` → `pnpm build` (Electron + Svelte, independent of server)

## Key Files

| File | Purpose |
|------|---------|
| `packages/shell-protocol/src/messages.ts` | Wire protocol schemas (Zod) - single source of truth |
| `packages/shell-protocol/src/client.ts` | `ShellClient` class + `IShellSocket` interface |
| `server/src/handlers/` | Server-side message handlers (session, I/O, workspace) |
| `web/src/components/Terminal.svelte` | xterm.js terminal widget (write buffering, auto-scroll) |
| `web/src/lib/workspaceStore.ts` | Split pane / tab workspace state |
| `web/src/lib/sessionContext.ts` | Svelte context API for session access |
| `tray/src/main/index.ts` | Tray app entry (Electron lifecycle, orchestration) |
| `tray/src/main/TrayManager.ts` | System tray icon + dynamic context menu |
| `npm/terminar/lib/server-manager.js` | Server process lifecycle (start/stop/status/logs) — shared by prod CLI and `scripts/dev.cjs` |
| `scripts/dev.cjs` | Dev orchestrator: builds server, starts it detached, runs UI in foreground |
| `docs/ARCHITECTURE.md` | Full architecture reference |
| `docs/API.md` | API reference documentation |

## Releasing

### npm Distribution Structure

The `terminar` CLI is published to npm as a multi-package setup:

- **`terminar`** (`npm/terminar/`) — Main package with CLI (`bin/terminar.js`), lib (`lib/`), and built Electron app (`app/`). Depends on `electron` and platform packages as `optionalDependencies`.
- **`@narai/terminar-darwin-arm64`** (`npm/darwin-arm64/`) — macOS ARM64 server binary
- **`@narai/terminar-darwin-x64`** (`npm/darwin-x64/`) — macOS x64 server binary
- **`@narai/terminar-linux-x64`** (`npm/linux-x64/`) — Linux x64 server binary
- **`@narai/terminar-linux-arm64`** (`npm/linux-arm64/`) — Linux ARM64 server binary
- **`@narai/terminar-win32-x64`** (`npm/win32-x64/`) — Windows x64 (deferred)

Users install with `npm install -g terminar`, which pulls the correct platform binary.

### Release Process

Releases are automated via `.github/workflows/release.yml`, triggered by `v*` tags:

1. **Bump versions** in all package.json files + Cargo.toml to the new version
2. **Commit and push**: `git push origin main`
3. **Tag and push**: `git tag v0.x.x && git push origin v0.x.x`

The workflow then:
1. Builds server binaries for all platforms (macOS arm64/x64, Linux x64/arm64)
2. Builds the tray Electron app (Vite only, not electron-builder)
3. Assembles npm packages (stamps versions, copies binaries + tray build)
4. Publishes platform packages to npm, then the main `terminar` package
5. Creates a GitHub Release with server binaries attached

**Requirements:**
- `NPM_TOKEN` GitHub secret (granular access token with bypass 2FA, read+write packages + narai org)
- Token expires every 90 days — regenerate at npmjs.com/settings/narayan-prem/tokens

### Version Locations (all must match for a release)

- `package.json` (root)
- `server/Cargo.toml`
- `tray/package.json`
- `web/package.json`
- `extension/package.json`
- `packages/shell-protocol/package.json`
- `npm/terminar/package.json` + `optionalDependencies` (stamped by CI)
- `npm/*/package.json` platform packages (stamped by CI)

## DO NOT MODIFY

- `node_modules/` - Dependencies
- `dist/` - Built output (terminar-protocol)
- `out/` - Compiled extension output
- `out-test/` - Compiled test output
- `target/` - Rust build output (cargo)
- `*.lock` files - Package lock files

---
> Source: [narailabs/terminar](https://github.com/narailabs/terminar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
