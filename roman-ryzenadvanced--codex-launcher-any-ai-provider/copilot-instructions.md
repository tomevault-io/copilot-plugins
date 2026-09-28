## codex-launcher-any-ai-provider

> OpenAI Codex CLI & Desktop launcher that proxies to **any** AI provider.

# Project: Codex Launcher — Any AI Provider

## Overview

OpenAI Codex CLI & Desktop launcher that proxies to **any** AI provider.
Python-only (stdlib), zero pip dependencies. Supports Responses API, Chat Completions,
Anthropic Messages API, Command Code, and more via a translation proxy.

Maintained by:

- **roman-ryzenadvanced** — original Linux development
- **cobra91** — Windows port (MSIX support)

## Architecture

```
codex-launcher-gui.py  (tkinter — cross-platform Linux + Windows)
  → codex_launcher_lib.py  (shared library: endpoints, config, process mgmt)
    → translate-proxy.py   (HTTP proxy: Responses API → backend API)
      → upstream provider (OpenAI, Anthropic, DeepSeek, Antigravity, etc.)
```

### Key Files

**Core (`src/`)** — required for the app to run:

| File                        | Purpose                                                |
| --------------------------- | ------------------------------------------------------ |
| `src/codex-launcher-gui.py` | tkinter GUI (cross-platform Linux + Windows)            |
| `src/codex_launcher_lib.py` | Shared library (endpoints, config, process management) |
| `src/translate-proxy.py` | Translation proxy (core routing, adapters, streaming) |
| `src/antigravity_grpc/` | gRPC client for Antigravity provider |
| `src/config_schema.py` | Configuration validation |
| `src/universal_runtime.py` | Cross-platform runtime utilities |
| `src/plugins/` | Plugin system for custom providers |
| `src/locales/` | i18n translation files |

**Tools (`tools/`)** — standalone utilities, not required for core operation:

| File                        | Purpose                                                |
| --------------------------- | ------------------------------------------------------ |
| `tools/codex-tui.py` | Curses TUI for headless/Termux |
| `tools/codex-dashboard.py` | Web health dashboard |
| `tools/codex-wizard.py` | Interactive setup wizard |
| `tools/codex-benchmark.py` | Provider latency benchmark |
| `tools/codex-health-monitor.py` | Background health monitoring |
| `tools/codex-backup.py` | Config backup/restore |
| `tools/cleanup-codex-stale.py` | Cross-platform stale process cleanup |
| `tools/cleanup-codex-stale.sh` | Linux stale process cleanup |
| `tools/mobile-control-panel.py` | HTTP panel for Android/Termux |

### Backend Types

| Type | Wire Protocol | Example |
|------|--------------|---------|
| `openai-compat` | Chat Completions | DeepSeek, OpenRouter, Crof.ai |
| `anthropic` | Anthropic Messages | Anthropic direct, OpenCode Zen |
| `command-code` | Command Code /alpha/generate | CommandCode API |
| `gemini-oauth-*` | Google OAuth | Google Antigravity |
| `codebuff`/`freebuff` | Agent-run lifecycle | Free DeepSeek/Kimi |

## Platform Compatibility

**MUST work on Linux, Windows, macOS, and Termux/Android.** No exceptions.

### Supported Platforms

| Platform | UI | Proxy | Installer |
|----------|-----|-------|-----------|
| Linux | GTK GUI, TUI, Dashboard | Full | `install.sh` |
| Windows | tkinter GUI | Full | `install.ps1` |
| macOS | CLI, TUI, Dashboard | Full | `install.sh` |
| Termux/Android | TUI, Dashboard, Mobile panel | Full | `install-termux.sh` |
| Docker | Dashboard, API | Full | `Dockerfile` |

### Platform-Specific Patterns

- **Process management**: `os.setsid()` + `os.killpg()` on Linux, `CREATE_NEW_PROCESS_GROUP` on Windows
- **Process listing**: `pgrep` on Linux, `tasklist` / `wmic` on Windows
- **Desktop launch**: exe path on Linux, `shell:AppsFolder\{AUMID}` for MSIX on Windows
- **Signals**: `signal.SIGTERM` on Linux, `taskkill /F` on Windows
- **Paths**: `~/.local/bin/` on Linux, `%LOCALAPPDATA%\Programs\Codex-Launcher\` on Windows
- **Config**: `~/.codex/config.toml` (same format on all platforms)
- **POSIX-only APIs**: `os.getpgid()`, `/proc/{pid}/stat`, `os.setsid()` — always guard with `sys.platform` checks
- **Termux**: `termux-wake-lock`, `termux-notification`, `termux-battery-status` — guard with `os.path.exists("/data/data/com.termux")`
- **curses**: Not available on Windows — TUI is Linux/macOS/Termux only

### Testing Cross-Platform

- Never assume Unix-only APIs exist (`pgrep`, `getpgid`, `SIGTERM`)
- Use `sys.platform == "win32"` for Windows branches
- Test proxy startup on both platforms before committing
- Provider presets (PROVIDER_PRESETS) work identically on all platforms

## Coding Conventions

- Python 3.8+ stdlib only, zero pip dependencies
- `snake_case` for functions/variables, `UPPER_CASE` for globals
- Immutable patterns: create new dicts/objects, don't mutate in-place
- Error handling: catch at boundaries, never silently swallow errors
- Thread-safe: use `threading.Lock` for shared state, `threading.Semaphore` for concurrency

## Common Pitfalls

- **MSIX exe paths**: `C:\Program Files\WindowsApps\` exes cannot be launched via `subprocess.Popen` — use `shell:AppsFolder` protocol
- **File locking on Windows**: Python can't overwrite files open in another process
- **Path separators**: always use `os.path.join()` or `Path` objects, never hardcoded `/`
- **Signal handling**: Windows doesn't support `SIGUSR1`/`SIGUSR2` — use events or named pipes
- **curses on Windows**: Not available — use `sys.platform` guard for TUI code
- **Termux paths**: `$PREFIX/bin` instead of `/usr/bin`, `$HOME/.codex` for config

## Testing

- **Run before every commit**: `python -m pytest tests/ -v`
- **All tests must pass** before pushing a PR
- Test files live in `tests/` directory
- Tests use `pytest` (not unittest runner)
- Platform-specific tests must skip gracefully on other OS: `pytest.mark.skipif(sys.platform != "linux", reason="Linux-only")`
- Never mock filesystem paths with hardcoded separators — use `os.path.join` or `tmp_path`

## CI/CD

- **GitHub Actions**: `.github/workflows/ci.yml` runs on push/PR
- **Linting**: `ruff check src/ tests/`
- **Format**: `ruff format --check src/ tests/`
- **Pre-commit**: `.pre-commit-config.yaml` configured with ruff

---
> Source: [roman-ryzenadvanced/Codex-Launcher-Any-AI-Provider](https://github.com/roman-ryzenadvanced/Codex-Launcher-Any-AI-Provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
