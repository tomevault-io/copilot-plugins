## beacon

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What this is

A dynamic API endpoint tester / load + rate-limit testing tool. The user defines endpoints (URL, method, headers, payload) and fires them repeatedly and/or concurrently, watching live stats (`attempts`, `success`, `rate_limited`, `errors`) and logs. Payloads/headers/URLs support `{{variable}}` templating with both static config variables and fresh-per-request generators. Intended for authorized testing of APIs (the seeded config targets `api.retailku.com`).

## Current implementation

Beacon's maintained application is the FastAPI + React desktop stack:

- `backend/app/main.py` — FastAPI backend on port 8000.
- `backend/app/core/` — request execution engine and its domain modules.
- `frontend/` — React/Vite/shadcn frontend on port 5173.

The old root-level Flask engine has been retired. Do not add a second copy of the execution engine; add reusable behavior to the focused modules under `backend/app/core/`.

## Running

### FastAPI backend (current)
```bash
cd backend
pip install -r requirements.txt
python -m uvicorn app.main:app --reload --port 8000
```

### React frontend (current)
```bash
cd frontend
npm install
npm run dev      # http://localhost:5173
npm run build    # tsc typecheck + vite build
```
The frontend hardcodes the backend at `http://localhost:8000` (see [frontend/src/App.tsx](frontend/src/App.tsx) and [frontend/src/components/EndpointEditor.tsx](frontend/src/components/EndpointEditor.tsx)). CORS in `main.py` only allows the `:5173` origin.

Python regression tests live in `backend/tests/` and can be run with `python -m pytest backend/tests` from the repository root after installing the backend requirements.

## Config persistence

The backend resolves its writable data directory through `backend/app/paths.py`, including the desktop sidecar/runtime locations. Do not introduce new cwd-relative persistence paths. Config remains the source of truth and mutations are persisted by the backend state/services layer.

## Core engine architecture (`backend/app/core/`)

- **`models.py`** — `EndpointTest` and `TestConfig`, including the persisted JSON contract.
- **`templating.py`** — recursive static-variable and fresh-per-request generator resolution.
- **`transport.py`** — construction and dispatch of HTTP requests for JSON, form, multipart, raw, and web targets.
- **`assertions.py`** — response assertion evaluation.
- **`extractors.py`** — response body/cookie extraction into runtime variables.
- **`metrics.py`** — thread-safe run counters, latency samples, and snapshots.
- **`tester.py`** — `APITester` orchestration and traffic-mode behavior. Keep this as the compatibility facade used by routers and MCP.

### Templating (`_substitute` / `_generate_dynamic`)
`{{...}}` tokens are resolved in two passes inside every string (and recursively through dicts/lists):
1. **Static** — `config.variables` keys are literal string-replaced first.
2. **Dynamic generators** — regex-matched and regenerated **fresh per request**: `random_email`, `random_phone` (hardcoded Indonesian `+62812…` format), `random_uuid`/`uuid`, `timestamp`, `random_string` / `random_string:<len>`, `random_number`/`random_int` / `random_int:<min>:<max>`. Unknown tokens are left intact as `{{token}}`.

### Extractors (token refresh)
After a successful (2xx) response, `extractors` (e.g. `{"access_token": "body.access_token"}`) pull values out of the JSON body (dot-path) or `Set-Cookie` header and **write them back into `config.variables`**, so chained runs (login → use token) stay fresh. This mutates shared config state at runtime.

### Success / rate-limit detection
Success = 2xx. Rate-limited = HTTP 429 **or** the response text containing "rate"/"too many" (substring heuristic).

## Run lifecycle

`POST /run` starts `APITester` execution and returns a `run_id`. Progress is exposed through the run status APIs and WebSocket events. Runs are held in runtime state and are lost on backend restart; durable summaries are written through the history layer.

## REST surface

Routes are grouped under `backend/app/routers/`. Keep HTTP validation and response shaping in routers, reusable application behavior in services, and request-execution mechanics in `core`.

## Security note
`config/tests.json` is operational state that can hold **real bearer tokens / JWTs and live target URLs**. Treat it as secret — do not commit real credentials to version control (`.gitignore` covers `config/*.local.json` but not `config/tests.json` itself).

---
> Source: [nannndev/beacon](https://github.com/nannndev/beacon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
