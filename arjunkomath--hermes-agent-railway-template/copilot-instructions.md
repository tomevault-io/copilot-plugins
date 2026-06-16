## hermes-agent-railway-template

> Python/Starlette web server that wraps Hermes Agent's gateway as a managed subprocess.

# Hermes Agent Railway Template

## Architecture

Python/Starlette web server that wraps Hermes Agent's gateway as a managed subprocess.

- `server.py` — Main server: HTTP handlers, gateway process manager, basic auth, .env file management
- `templates/index.html` — Single-page UI with Tailwind CSS + Alpine.js
- Config is stored as a flat `.env` file at `/data/.hermes/.env` (Hermes uses python-dotenv)
- Gateway is spawned via `hermes gateway` command with env vars from the .env file

## Key patterns

- Gateway lifecycle: start/stop/restart via async subprocess, stdout captured to ring buffer
- Secret masking: password fields show first 8 chars + `***`, merge on save preserves masked values
- No direct Hermes Python imports — the server manages the .env file independently
- Auto-start: gateway starts on server boot if any provider API key is configured

---
> Source: [arjunkomath/hermes-agent-railway-template](https://github.com/arjunkomath/hermes-agent-railway-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
