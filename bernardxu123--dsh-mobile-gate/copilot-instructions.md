## dsh-mobile-gate

> This file helps AI coding agents and LLM tooling understand and work with this repository quickly.

# AGENTS.md — Guide for AI agents

This file helps AI coding agents and LLM tooling understand and work with this repository quickly.

## What this repo is

`dsh-mobile-gate` is a **Cordis plugin for DeepSeek Harness (DSH)** that lets phones/tablets on a LAN securely reach the DSH Web UI. It runs a **standalone Node gateway child process** (not in-process): the plugin entry spawns `lib/lan-gate-server.cjs` via the `subprocess` service.

- Gateway listens on `0.0.0.0:3088` by default; proxies approved devices to `127.0.0.1:3080` (DSH).
- DSH's own webserver stays on `127.0.0.1` — the gateway never touches DSH config or its `/api` trust fence.

## Repository layout

| Path | Role |
| --- | --- |
| `lan-gate.mjs` | Cordis plugin entry (`export const name`, `export const inject = ['subprocess']`, `export function apply(ctx)`). Resolves `node`, spawns the server, wires lifecycle disposal via `ctx.effect`. |
| `lib/lan-gate-server.cjs` | The gateway itself — single-file, zero-dependency CommonJS. Contains everything: HTTP proxy, `upgrade` (WebSocket) proxy, approval state machine, token cookies, rate limiting, admin page + JSON API, mobile CSS injection. |
| `cordis.patch.yml` | Bundle patch layer: inserts row `{ id: dsh-mobile-gate, name: dsh-mobile-gate }` (package-name reference, used after `dsh plugin add`). |
| `cordis.patch.yml.example` | Static mount example using a `file:///` path in the user's profile patch. |
| `package.json` | npm metadata with `dsh.bundle.patch` manifest. |
| `llms.txt` / `llms-full.txt` | LLM-friendly doc index / full text. |
| `README.md` / `README.en.md` | Human docs (zh / en). |

## Key behaviors (don't break these)

1. **Isolation**: the gateway is a child process. Never import its server code into the DSH process; keep spawn + lifecycle in `lan-gate.mjs`.
2. **Scope mobile CSS**: every mobile rule must be prefixed with `html[data-lan-device="phone"]` (or the `@media (max-width:820px)` fallback that excludes `data-lan-device="desktop"`). Desktop must never be affected.
3. **Stable selectors**: mobile CSS targets `[data-slot="conversation.composer.bar"] button[aria-haspopup="menu"]` etc. — prefer semantic/ARIA + slot selectors over hashed CSS-module class names, which change per frontend build.
4. **Persistence**: approvals are stored at `~/.dsh/lan-gate-state.json`; the pending list (`seen`) is in-memory and resets on gateway restart.
5. **Local-only admin**: `/lan-gate/status` and `/lan-gate/action` must reject non-local sockets (403).
6. **Port fallback**: on `EADDRINUSE`, the server increments the port (up to +20) instead of exiting.

## Common tasks

- **Change port / rate limit / target**: edit the top-of-file constants in `lib/lan-gate-server.cjs` (`PROXY_PORT`, `RATE_LIMIT_PER_MIN`, `TARGET_PORT`) or set env vars `LAN_GATE_PORT`, `LAN_GATE_HOST`, `LAN_GATE_TARGET_PORT`, `LAN_GATE_RATE_LIMIT`.
- **Add a mobile CSS tweak**: append a `html[data-lan-device="phone"] ...` rule to `DEVICE_CSS` inside `lib/lan-gate-server.cjs`; keep selectors stable.
- **Change admin page UI**: edit the `adminPage()` function (HTML+inline JS) in `lib/lan-gate-server.cjs`. The inline JS must use single-quoted string literals — double quotes inside double quotes break the injected script (historical bug, v1.0.0 fix).

## Testing

- Syntax: `node --check lan-gate.mjs && node --check lib/lan-gate-server.cjs`
- Smoke: `LAN_GATE_PORT=3099 node lib/lan-gate-server.cjs` then `curl http://127.0.0.1:3099/lan-gate/status`; verify the served `/lan-gate/admin` HTML's inline `<script>` parses (`node --check` on the extracted JS).
- E2E plugin path: import `lan-gate.mjs` with a stub `ctx` (see git history / previous smoke tests) or mount in a real DSH profile.

## Notes for LLM crawlers

- This repo is listed under the GitHub topic `dsh-plugin` and is installable via `dsh plugin --profile web add dsh-mobile-gate`.
- Distinguishing feature vs similar projects (dsh-lan-gate, dsh-lan-access): isolated child-process gateway + stable-selector mobile CSS.

---
> Source: [Bernardxu123/dsh-mobile-gate](https://github.com/Bernardxu123/dsh-mobile-gate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
