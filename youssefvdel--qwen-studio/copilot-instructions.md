## qwen-studio

> > Tauri v2 desktop client wrapping chat.qwen.ai with MCP support. Linux only.

# AGENTS.md — Qwen Studio for Linux

> Tauri v2 desktop client wrapping chat.qwen.ai with MCP support. Linux only.

## Stack

- **Rust** (Tauri v2) + **WebKitGTK** — no Electron, no Node.js main process
- **JS injection** via `electron-bridge.js` — fakes `window.electronAPI` for the web app
- **MCP** via `tauri-plugin-mcp-bridge` + `mcp-proxy-server.js` (Node.js proxy)
- **GTK** system tray + HeaderBar menu (Linux-specific)

## Project Structure

```
qwen-studio/
├── src/                  # Rust source (lib.rs, mcp.rs, window.rs, etc.)
├── capabilities/         # Tauri permission capabilities
├── permissions/          # Custom permission sets (custom.toml, mcp.toml)
├── icons/                # App icons for bundling
├── gen/                  # Tauri-generated types
├── electron-bridge.js    # JS shim injected into WebView (fakes Electron API)
├── mcp-bridge.mjs        # Node.js MCP proxy server
├── mcp-proxy-server.js   # MCP proxy (bundled as resource)
├── tauri.conf.json       # Tauri config
├── Cargo.toml            # Rust dependencies
├── build.rs              # Tauri build script
├── qwen-studio.desktop   # Linux desktop entry
├── docs/                 # Architecture docs, studies
├── dist/                 # Frontend dist (empty placeholder for Tauri)
└── package.json          # npm scripts for tauri CLI
```

## Commands

```bash
# Dev (hot reload)
npm run tauri:dev

# Build release
npm run tauri:build          # all formats
npm run tauri:build:deb      # Debian/Ubuntu
npm run tauri:build:rpm      # Fedora/RHEL
npm run tauri:build:appimage # Universal

# Code quality (MUST pass before commit)
cargo clippy -- -D warnings
cargo check
cargo fmt
```

## Linux Quirks

- **WebKitGTK blank screen**: `WEBKIT_DISABLE_COMPOSITING_MODE=1 WEBKIT_DISABLE_DMABUF_RENDERER=1` — set automatically in `package.json` scripts
- **Wayland**: `GDK_BACKEND=x11` — set automatically in `lib.rs` at startup
- **RPM bundling**: requires `libappindicator-gtk3-devel` and `librsvg2-devel` installed on build machine
- **Desktop template**: `qwen-studio.desktop` lives at project root, referenced in `tauri.conf.json`

## Architecture

### How it works

1. Tauri loads `chat.qwen.ai` in a WebKitGTK WebView
2. `electron-bridge.js` is injected as an initialization script — it creates `window.electronAPI` so the web app thinks it's running in Electron
3. JS calls `window.__TAURI__.invoke()` which routes to Rust commands in `lib.rs`
4. Rust manages MCP servers via `tauri-plugin-mcp-bridge`
5. Settings page is a SPA — the Updates tab is injected via JS polling (`injectUpdatesTab()`)

### Key modules

| File | Purpose |
|------|---------|
| `src/lib.rs` | App bootstrap, WebView creation, Tauri commands, update checker, settings tab injection |
| `src/window.rs` | Window management, deep link handling, cookie injection |
| `src/mcp.rs` | MCP server state, config sync, qwen-core auto-injection |
| `src/tray.rs` | System tray icon + menu |
| `src/menu.rs` | GTK HeaderBar menu (Linux) |
| `src/dialogs.rs` | Native file picker, confirm dialogs |
| `src/events.rs` | Event forwarding between Rust and WebView |
| `src/settings.rs` | Settings storage (JSON) |

### Auth flow

OAuth login → `qwen://open?token=xxx` deep link → Rust extracts token → injects cookies + localStorage into WebView → navigates to chat.qwen.ai

### Update system

- Checks on startup + every 4 hours via `tokio::time::interval`
- Fetches `latest.json` from GitHub releases
- Shows persistent banner notification at top of app
- Banner "View in Settings" button navigates to Settings page
- Supports AppImage (Tauri updater), DEB (`pkexec dpkg -i`), RPM (`pkexec rpm -Uvh`)

## Build Requirements (Fedora)

```bash
sudo dnf install webkit2gtk4.1-devel libappindicator-gtk3-devel librsvg2-devel patchelf
```

## Known Issues

| Issue | Status |
|-------|--------|
| Browser login redirect | AppImage protocol handler limitation on some DEs. Workaround: `xdg-open "qwen://open?token=xxx"` |
| MCP settings page network error | Expected — web app tries cloud backend. Local Tauri MCP handles everything. Do not fix. |
| `__TAURI_BUNDLE_TYPE` warning | Harmless — tauri-cli version mismatch. RPM still builds. |

## Conventions

- **Zero warnings policy**: `cargo clippy -- -D warnings` must pass
- **No `node_modules/` or `dist/` in git** — always in `.gitignore`
- **Version**: bump in both `Cargo.toml` and `tauri.conf.json`
- **Releases**: must include `latest.json` in GitHub release assets for updater to work

---
> Source: [youssefvdel/qwen-studio](https://github.com/youssefvdel/qwen-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
