## ssh-agent-proxy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
go build ./...
go build -o ssh-agent-proxy .
go run . service run -config ~/.config/ssh-agent-proxy/config.json
```

Sub-commands:
- `service run [flags]` — run the proxy agent (foreground). Flags: `-config`, `-listen`, `-log`, `-debug`, `-version`
- `service install [-config path]`, `service uninstall`, `service start`, `service stop` — manage system service (macOS launchd / Linux systemd user service; stub on other platforms).

Send `SIGUSR1` to invalidate all upstream key caches at runtime.

## Architecture

This is a single-binary Go SSH agent proxy that multiplexes multiple upstream SSH agents behind one Unix socket.

**Data flow:**
1. `main.go` — accepts connections on a Unix socket, spawns `handleConn` per client
2. `proxy.go` — `ProxyAgent` implements `agent.ExtendedAgent`; fans out `List()` across all upstreams in parallel (via `errgroup`), maintains a `keyMap` mapping key bytes → upstream, routes `Sign`/`SignWithFlags` to the correct upstream
3. `upstream.go` — `UpstreamAgent` wraps a single upstream socket with optional key-list caching (`none` / `duration` / `forever`)
4. `peercred_darwin.go` / `peercred_linux.go` — platform-specific `getPeerCred()` to log connecting process PID/UID/GID/cmdline

**Key design points:**
- Only `List` and `Sign`/`SignWithFlags` are forwarded; `Add`, `Remove`, `Lock`, etc. return `errNotSupported`
- Duplicate keys across upstreams: first upstream wins (logged as warning)
- Unknown keys invalidate all upstream caches and trigger a full `List()` refresh before failing
- Config at `~/.config/ssh-agent-proxy/config.json` (JSON); auto-created with defaults on first run

## Config Structure

```json
{
  "listen": "/tmp/ssh-agent-proxy.sock",
  "upstreams": [
    {"name": "system", "socket": "$SSH_AUTH_SOCK", "cache": "none"},
    {"name": "hw-key", "socket": "~/.gnupg/S.gpg-agent.ssh", "cache": "duration", "cache_duration": "5m"}
  ],
  "log": {"enabled": true, "file": "/tmp/ssh-agent-proxy.log", "level": "info"}
}
```

Cache policies: `none`, `duration` (requires `cache_duration` as Go duration string), `forever`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Inndy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
