## flrig-sandbox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies into the local venv
.venv/bin/pip install -r requirements.txt

# Run the server (development, auto-reload)
.venv/bin/uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Run with custom flrig host/port
FLRIG_HOST=192.168.1.10 FLRIG_PORT=12345 .venv/bin/uvicorn main:app --reload

# Test flrig XML-RPC connectivity directly (no server required)
.venv/bin/python3 -c "
import xmlrpc.client; p = xmlrpc.client.ServerProxy('http://localhost:12345')
print('vfo:', p.rig.get_vfo()); print('xcvr:', p.rig.get_xcvr())
"

# Interactive API docs (when server is running)
# http://localhost:8000/docs

# Run the mock flrig XML-RPC server (port 12345) — no radio required
.venv/bin/python3 mock_flrig.py

# Mock on a different port
.venv/bin/python3 mock_flrig.py --host localhost --port 12346
```

### curl examples

```bash
curl http://localhost:8000/vfo
curl -X POST http://localhost:8000/vfo -H "Content-Type: application/json" -d '{"frequency_hz": 14225000}'
curl http://localhost:8000/health
```

## Architecture

### Files

| File | Purpose |
|---|---|
| `main.py` | FastAPI REST-to-XML-RPC bridge (port 8000) |
| `mock_flrig.py` | Standalone mock flrig XML-RPC server (port 12345) |

This is a single-file FastAPI app (`main.py`) that acts as a REST-to-XML-RPC bridge between HTTP clients and the **flrig** ham radio control daemon.

**Request flow:**
```
curl / HTTP client  →  FastAPI (port 8000)  →  xmlrpc.client  →  flrig (port 12345)
```

**Key design points:**

- `flrig()` is a context manager (not a class) that creates a fresh `xmlrpc.client.ServerProxy` per request and translates `xmlrpc.client.Fault` and `OSError` into `HTTP 502` responses. All endpoint handlers use `with flrig() as rig:` to get the proxy.
- flrig's XML-RPC methods are namespaced under `rig.*` (e.g. `rig.get_vfo`, `rig.set_vfo`). The Python call is therefore `rig.rig.method_name()` — the first `rig` is the local proxy variable, the second is the XML-RPC namespace.
- `POST /vfo` reads the frequency back after setting it, so the response always reflects what the radio actually accepted.
- flrig connection target is configured via `FLRIG_HOST` / `FLRIG_PORT` env vars (defaults: `localhost:12345`).

## flrig behaviour notes

- All `set_*` XML-RPC methods return `0` on failure and a non-zero value on success.
- When no transceiver is physically connected to flrig, `rig.get_xcvr()` returns an empty string and all `set_*` calls silently return `0` without changing state. `get_vfo` will still return the last known frequency held in flrig's memory. This is normal — the REST API is working correctly even when the radio is absent.
- `POST /vfo` confirms the result by reading back the frequency after setting it, so the response body always reflects what the radio actually accepted rather than what was requested.

## Adding new flrig commands

1. Find the XML-RPC method name via `GET /health` (lists all methods flrig exposes).
2. Add a Pydantic model for any request body.
3. Call `rig.<namespace>.<method>()` inside a `with flrig() as rig:` block.
4. Return a typed Pydantic response model.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stevenmburns)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/stevenmburns)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
