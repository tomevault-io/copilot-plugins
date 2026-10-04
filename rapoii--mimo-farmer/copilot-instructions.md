## mimo-farmer

> mimo-farmer is a Python CLI tool for automated Xiaomi MiMo account creation. We're adding a self-hosted web UI that runs on localhost.

# mimo-farmer Web UI — Project Instructions

## Project Context

mimo-farmer is a Python CLI tool for automated Xiaomi MiMo account creation. We're adding a self-hosted web UI that runs on localhost.

## Architecture

- **Backend**: FastAPI + uvicorn (Python) — serves API + static frontend files
- **Frontend**: Single-page HTML/CSS/JS (no build step, no Node.js) with neobrutalism theme
- **Real-time**: WebSocket for live progress updates during account creation
- **Data**: Parses existing `accounts/batch_*.txt` files (no database)
- **Integration**: Wraps existing `mimo_farmer/` modules (creator.py, cli.py, etc.)

## Entry Point

```bash
mimo web [--port 8080] [--host 127.0.0.1]
```

This starts FastAPI server that serves both the API and the static frontend.

## Directory Structure

```
mimo_farmer/
├── web/
│   ├── __init__.py
│   ├── server.py         # FastAPI app + routes + WebSocket
│   ├── api.py            # API endpoint handlers
│   ├── ws_manager.py     # WebSocket connection manager
│   ├── batch_parser.py   # Parse batch_*.txt files
│   └── static/
│       ├── index.html    # Single-page app
│       ├── style.css     # Neobrutalism theme
│       └── app.js        # Frontend logic + WebSocket client
```

## Key Files to Read

- `docs/web/PRD.md` — Product requirements, features, API endpoints
- `docs/web/DCD.md` — Context diagram (system boundaries)
- `docs/web/DFD.md` — Data flow diagram (processes + data movement)
- `docs/web/ERD.md` — Entity relationship diagram (data model)
- `docs/web/DESIGN.md` — Neobrutalism design system (colors, typography, components)
- `mimo_farmer/cli.py` — Existing CLI (reference for modes, args, batch parsing)
- `mimo_farmer/creator.py` — Account creation pipeline (14 steps)
- `mimo_farmer/config.py` — Default settings
- `mimo_farmer/captcha.py` — CAPTCHA handling

## API Endpoints (from PRD)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Serve frontend (static index.html) |
| GET | `/api/stats` | Dashboard statistics |
| GET | `/api/accounts` | List all accounts (paginated, query params for filter/search) |
| GET | `/api/batches` | List batch files with metadata |
| POST | `/api/create` | Start account creation job |
| POST | `/api/create/cancel` | Cancel running job |
| WS | `/ws/progress` | Real-time progress updates |
| GET | `/api/export?format=json\|txt\|csv` | Export accounts |
| GET | `/api/settings` | Get current settings |
| PUT | `/api/settings` | Update settings |

## WebSocket Messages (Server → Client)

```json
{
  "type": "progress",
  "data": {
    "job_id": "uuid",
    "step": 3,
    "step_name": "reCAPTCHA",
    "status": "running",
    "message": "Solving audio challenge...",
    "progress_pct": 21,
    "log": "[3/14] Downloading captcha audio..."
  }
}
```

```json
{
  "type": "account_created",
  "data": {
    "email": "user@domain.com",
    "balance": "$2.72",
    "api_key": "sk-abc...xyz"
  }
}
```

```json
{
  "type": "job_complete",
  "data": {
    "job_id": "uuid",
    "total_created": 6,
    "total_failed": 0
  }
}
```

## Design Rules (from DESIGN.md)

1. **Neobrutalism theme** — thick 3px black borders, solid shadows (no blur), bold colors
2. **Colors**: bg=#f5f0e8, main=#5b8cff, secondary=#ffb938, border=#1a1a1a
3. **Shadows**: `5px 5px 0px #1a1a1a` (translate on hover, none on active)
4. **Typography**: Space Grotesk (headings), Inter (body), JetBrains Mono (code)
5. **Grid background**: subtle 20px grid pattern
6. **No build step** — pure HTML/CSS/JS served as static files

## Critical Constraints

1. **Playwright runs in background thread** — must NOT block the FastAPI event loop
2. **Batch files are the source of truth** — parse from disk, don't cache in memory
3. **Manual CAPTCHA flow** — when Xiaomi text CAPTCHA detected, send WebSocket message to frontend, show "Solve in browser" prompt, wait for user
4. **No database** — all state comes from files or is ephemeral (in-memory during job)
5. **Existing modules** — import and use `mimo_farmer.creator`, `mimo_farmer.config`, etc. Don't recreate logic.
6. **CLI still works** — web is additive, don't break existing `mimo create` flow

## Implementation Order

1. Backend: `server.py` (FastAPI app, static file serving, WebSocket endpoint)
2. Backend: `api.py` (all API handlers)
3. Backend: `ws_manager.py` (WebSocket broadcast)
4. Backend: `batch_parser.py` (parse batch files → account list)
5. Frontend: `style.css` (full neobrutalism design system)
6. Frontend: `index.html` (SPA structure with all pages)
7. Frontend: `app.js` (routing, API calls, WebSocket client, DOM manipulation)
8. CLI integration: add `mimo web` command to `cli.py`
9. Test: run `mimo web`, verify all pages work

---
> Source: [rapoii/mimo-farmer](https://github.com/rapoii/mimo-farmer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
