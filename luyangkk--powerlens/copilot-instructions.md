## powerlens

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PowerLens is a macOS-only Oh-My-Zsh plugin that embeds live system metrics (power, battery, CPU, CPU temp, fan speed, memory, network) into zsh `RPROMPT`. It has two layers:

1. **Go daemon** (`src/`) — a compiled binary that collects metrics via macOS-native APIs and writes a JSON cache file every N seconds.
2. **Zsh plugin** (`powerlens.zsh` / `powerlens.plugin.zsh`) — reads the cached JSON in `precmd` and formats `RPROMPT`.

## Build Commands

```bash
make arm64      # Build for Apple Silicon → bin/powerlens-fetch-arm64
make amd64      # Build for Intel → bin/powerlens-fetch-amd64
make all        # Build both
make clean      # Remove binaries
```

Build requires CGO (`CGO_ENABLED=1`) and links against macOS `IOKit` and `CoreFoundation`. Each binary is ad-hoc codesigned automatically. Cross-arch compilation is not straightforward due to CGO.

## Tests

```bash
# Go unit tests (requires macOS — IOKit/SMC collectors are unavailable on Linux)
cd src && go test ./...

# Go vet
cd src && go vet ./...

# Zsh integration tests
zsh tests/test_plugin.zsh
```

CI runs `go vet ./...` + `go test ./...` on `macos-latest`.

## Architecture

### Data flow

```
precmd / TRAPALRM (zsh)
    → read ~/.cache/powerlens/metrics.json (only on mtime change)
    → format RPROMPT

powerlens-fetch --daemon (Go binary, singleton via PID file)
    → collects all metrics every POWERLENS_REFRESH seconds
    → writes ~/.cache/powerlens/metrics.json atomically (write-to-tmp, rename)
```

Cache dir is `$XDG_CACHE_HOME/powerlens` or `~/.cache/powerlens`. The daemon singleton is managed via `daemon.pid` and a `sessions` reference counter — the last shell to exit kills the daemon.

### Go `src/collect/` package

| File | Responsibility |
|---|---|
| `metrics.go` | `Metrics` struct, `All()` aggregator, `WriteJSON()`, `XDGCacheDir()` |
| `power_darwin.go` | Power (W) via `powermetrics` plist + fallback `ioreg`; battery % + charging via IOKit CGO |
| `temp_darwin.go` | CPU temp via `powermetrics` plist (Apple Silicon) or SMC read via IOKit CGO (Intel) |
| `fan_darwin.go` | Fan speed (RPM avg) via SMC IOKit CGO; reads `FNum`/`F0Ac`/`F1Ac`…; returns -1 on fanless/unavailable |
| `cpu.go` | CPU % via `gopsutil` |
| `mem.go` | Memory % via `gopsutil` |
| `net.go` | Network MB/s via `gopsutil`; interface resolution (`default`/`wifi`/`ethernet`) via `route get default` + `networksetup` |

`main.go` accepts `--daemon`, `--iface`, `--refresh` flags. Without `--daemon` it runs one-shot and prints JSON to stdout (useful for debugging).

### Zsh plugin

`powerlens.plugin.zsh` sets all `POWERLENS_*` defaults (with `${VAR:=default}` so user values set before `plugins=()` take precedence), then sources `powerlens.zsh`.

`powerlens.zsh` contains all logic:
- `_powerlens_jget` — regex-based JSON field extractor (no `jq` dependency)
- `_powerlens_color` — maps metric + value → hex color, supports `multi` (4-level gradient) and `alert` (gray/orange) modes
- `_powerlens_format` — assembles the full `RPROMPT` string from cached JSON
- `_powerlens_update_rprompt` — called by `precmd`; skips re-parse if `mtime` unchanged; auto-restarts daemon if data is stale (>10s)
- `TRAPALRM` / `TMOUT` — triggers periodic in-place `RPROMPT` refresh while ZLE is active (without waiting for Enter)

## Key Constraints

- **macOS only.** CGO links against `IOKit`/`CoreFoundation`; `powermetrics` and `networksetup` are macOS-specific.
- **No new Go dependencies without discussion** — the binary size affects install time.
- **Go formatting**: standard `gofmt`; enforced by CI.
- **Zsh formatting**: match the style in `powerlens.zsh` (function-per-concern, `local` variables).
- The Go module path is `github.com/user/powerlens` (see `src/go.mod`).

---
> Source: [luyangkk/powerlens](https://github.com/luyangkk/powerlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
