## lan-proxy-gateway

> Generates mihomo's `config.yaml` from an embedded template (`embed/embed.go`) by replacing `{{PLACEHOLDER}}` tokens. After basic rendering, extension modes mutate the output:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`lan-proxy-gateway` is a CLI tool (`gateway`) that turns a Mac/Linux/Windows machine into a LAN transparent proxy gateway, built on top of [mihomo](https://github.com/MetaCubeX/mihomo). It handles IP forwarding, TUN mode, rule-based traffic routing, and chain proxying for AI services.

## Build & Development

```bash
# Build binary
make build

# Install to /usr/local/bin/gateway (requires sudo)
make install

# Cross-platform release builds (darwin/linux/windows)
make build-all

# Run all tests
go test ./...

# Run a single package's tests
go test ./cmd/...
go test ./internal/config/...

# Run a specific test
go test ./internal/config/ -run TestConfigLoad
```

The binary is built with version ldflags: `go build -ldflags "-s -w -X main.version=$(VERSION)" -o gateway .`

## Architecture

### Entry point & CLI
- `main.go` — sets version (injected via ldflags) and calls `cmd.Execute()`
- `cmd/root.go` — Cobra root command; `cmd/root_unix.go` / `cmd/root_windows.go` — platform-specific root setup
- Each subcommand is a separate file: `cmd/start.go`, `cmd/stop.go`, `cmd/install.go`, `cmd/status.go`, etc.

### Startup flow (`cmd/start.go`)
`runStart` → `runStartWithMode`:
1. Load `gateway.yaml` config
2. Generate `config.yaml` for mihomo via template rendering
3. Enable IP forwarding (`internal/platform`)
4. Start mihomo process
5. Enter interactive console loop (simple mode by default, TUI mode with `--tui`)

### Console modes
Two distinct console modes, both entered via `runInteractiveConsoleLoop`:
- **Simple mode** (`cmd/start.go:runSimpleRuntimeConsole`): line-based REPL using `bufio.Reader`. Command dispatch is split between `cmd/simple_console_actions.go` (config commands like `tun`, `proxy`, `rule`, `chain`, `extension`) and `cmd/start.go` (runtime commands like `status`, `restart`, `stop`).
- **TUI mode** (`cmd/runtime_console.go`): full Bubble Tea TUI with tabs (Dashboard / Proxy / Subscription), built with `charm.land/bubbletea/v2` and `charm.land/lipgloss/v2`.

### Config (`internal/config/config.go`)
`gateway.yaml` is the user config. On load, it is deserialized into `Config` struct with legacy field migration handled by `diskConfig.toConfig()`. Key sections:
- `proxy` — subscription source (url/file), profiles
- `runtime` — ports (mixed:7890, redir:7892, api:9090, dns:53), TUN settings
- `rules` — toggleable rule sets (lan_direct, china_direct, apple_rules, etc.)
- `extension` — mode (`chains`/`script`/empty), residential chain config

### Template rendering (`internal/template/render.go`)
Generates mihomo's `config.yaml` from an embedded template (`embed/embed.go`) by replacing `{{PLACEHOLDER}}` tokens. After basic rendering, extension modes mutate the output:
- `chains` mode: `internal/script/chains.go` injects residential proxy YAML
- `script` mode: `internal/script/runner.go` executes a JS file via `goja`

### Rules (`internal/rules/`)
- `base.go` — renders YAML rule blocks based on `RulesConfig` toggles
- `chain.go` — additional rules specific to chains mode

### Platform abstraction (`internal/platform/`)
Per-OS implementations of: IP forwarding, TUN interface detection, process start/stop, network interface detection, firewall config, service installation. Files follow `_darwin`, `_linux`, `_windows` naming.

### mihomo API client (`internal/mihomo/api.go`)
HTTP client wrapping the mihomo REST API (proxy group listing, node selection, traffic stats). Used by TUI dashboard and status commands.

### Config file location
Runtime data (generated `config.yaml`, proxy providers, logs) lives in `~/.config/lan-proxy-gateway/` (or platform equivalent). User config is `~/.config/lan-proxy-gateway/gateway.yaml`.

## Key Patterns

- **Workspace context in simple mode**: `simpleWorkspace` enum in `simple_console_actions.go` tracks which sub-menu the user is in, enabling numeric shortcuts (e.g., `1` → `tun toggle` when in runtime workspace).
- **Config mutations**: All config changes go through helper functions (e.g., `updateTunEnabled`, `updateProxySource`) that load, mutate, and save `gateway.yaml`, then return the updated config for immediate display.
- **Platform builds**: Use `//go:build` tags; `cmd/console_windows.go` vs no equivalent unix file. Platform-specific `internal/platform/` files selected by Go's standard OS suffix convention.

---
> Source: [Tght1211/lan-proxy-gateway](https://github.com/Tght1211/lan-proxy-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
