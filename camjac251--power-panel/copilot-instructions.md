## power-panel

> Remote server power management via Redfish/WoL. Go + templ + htmx + SSE + Tailwind v4 + templUI.

# Power Panel

Remote server power management via Redfish/WoL. Go + templ + htmx + SSE + Tailwind v4 + templUI.

## Commands

```bash
mise run setup       # Install Go dependencies
mise run dev         # Hot-reload dev server (templ + air + tailwindcss watchers)
mise run build       # Production build (generate + css + compile)
mise run build:arm64 # Cross-compile for ARM64
mise run deploy      # Deploy to target host via SSH
mise run test        # go test ./...
mise run test:race   # go test -race ./...
mise run lint        # golangci-lint
mise run fmt         # Auto-format (gofumpt + goimports)
```

Dev server requires `IPMI_PASS` env var (auto-loaded from `.env` via mise) and a `dev-config.yaml` (both gitignored):
```bash
./tmp/main -config dev-config.yaml -addr 0.0.0.0:8080
```

## Architecture

Single-binary Go app. Page loads instantly (no BMC blocking), SSE fills in live data.

```
main.go              Entry point, flag parsing, wiring
internal/
  bmc/               Redfish client (gofish), WoL, TCP reachability check
  config/            YAML config + IPMI_PASS from env
  db/                SQLite (modernc, pure Go) - power event audit log
  server/            HTTP routes, SSE handler, power action handlers
  updater/           Self-update via GitHub Releases + digest verification
views/               templ templates (home, layout, update, helpers)
components/          templUI components (button, dialog, icon, popover, toast, tooltip)
utils/               templUI utility (TwMerge, ComponentScript)
assets/              Embedded via go:embed (fonts, JS, generated CSS)
deploy/              systemd service + example config
input.css            Tailwind v4 source (theme, fonts, custom utilities)
```

## Key Patterns

- **templ code generation**: `.templ` files generate `_templ.go` (gitignored). Run `templ generate` before `go build`.
- **Tailwind v4**: Uses `@import "tailwindcss"` + `@theme` block in `input.css`. `@utility` for custom classes. No tailwind.config.js.
- **No node_modules**: mise installs `@tailwindcss/cli`. The v4 CLI resolves its own imports internally.
- **templUI components**: Installed via `templui add`. Config in `.templui.json`. JS assets in `assets/js/`.
- **Icons**: Use `icon.IconName()` from `components/icon` (Lucide SVGs, cached in memory). Never inline SVGs.
- **BMC cache**: `bmc.Cache` deduplicates BMC requests from concurrent SSE clients. One BMC call per poll interval regardless of client count. Invalidated after power actions. SSE reads from cache; API endpoints hit BMC directly.
- **Tailscale auth**: Identity headers (`Tailscale-User-Login`) trusted only from loopback (Tailscale Serve proxy). Non-local requests use the WhoIs unix socket. No login page needed.
- **Auto-updater**: Checks GitHub Releases every 6h (1 min delay after startup). Downloads asset, verifies SHA256 against GitHub's immutable asset digest API field, replaces binary via `update.Apply` (atomic rename with rollback). Exits cleanly for systemd restart. Disabled in containers (Docker/Podman) and dev mode (`version=dev`). Configurable via `update.enabled` and `update.auto_apply` in config.yaml.

## Prerequisites

mise (Go, templ, tailwindcss, air), a Redfish-capable BMC, `IPMI_PASS` env var. Tailscale for auth in production.

## SSE + htmx

SSE pushes 5 events: `status` (badge), `power` (button), `actions` (action buttons), `sensors-key`, `sensors-all`. Clients connect via `hx-ext="sse" sse-connect="/api/sse"`.

Sensors use dedicated SSE swap targets to avoid resetting the "All sensors" details toggle. Events use htmx polling.

## Gotchas

- **Dialogs outside SSE targets**: Dialog JS moves content to `document.body`. Rendering dialogs inside SSE swap regions causes DOM leaks (duplicated hx-post elements). Render dialog definitions once outside the SSE div, use `dialog.TriggerProps{For: "id"}` for buttons inside swap targets.
- **gofish ConnectContext**: `ConnectContext()` cancels its context before the session is used. The code uses `Connect()` instead. Do not switch to `ConnectContext()` for persistent sessions.
- **BMC sensors null when off**: Redfish sensors return null values when the host is powered off. Filter zero/null readings in templates.
- **NCSI NIC drops**: The BMC shares its NIC with the host via NCSI. During power transitions, the BMC may become temporarily unreachable. SSE treats BMC-unreachable during shutdown as confirmation.
- **BMC ResetTypes**: The target BMC supports On, ForceOff, ForceRestart, GracefulShutdown. It rejects PowerCycle and GracefulRestart.
- **Transition states**: PoweringOn/ShuttingDown/Restarting are set by the server (not BMC) for immediate UI feedback. Auto-clear when BMC confirms or after 2 min timeout.
- **Ping host in dev**: TCP port 22 check for OS reachability. Disabled in WSL dev (can't route to server VLAN). Enable via `server.ping_host` in config.
- **GitHub asset digests**: The updater verifies downloads against GitHub's server-computed SHA256 digest (available since June 2025 on the releases API). No checksums.txt file needed. The digest is immutable and computed at upload time.

---
> Source: [camjac251/power-panel](https://github.com/camjac251/power-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
