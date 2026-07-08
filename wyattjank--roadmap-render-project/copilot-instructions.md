## roadmap-render-project

> Guidance for AI agents working in this repository.

# AGENTS.md

Guidance for AI agents working in this repository.

## Project overview

Roadmap Render Project: a FastAPI admin API + React UI for editing release roadmaps (CSV on disk), plus a CLI (`render.py`) for Excel/draw.io export.

## Cursor Cloud specific instructions

### Services

| Service | Port | Required for web E2E |
|---------|------|----------------------|
| Uvicorn (`web.app:app`) | 8080 | Yes |
| Vite dev server (`npm run dev`) | 5173 | No (optional for frontend hot reload) |

There is no database, Docker Compose stack, or separate backend process. Data lives in CSV/JSON files at the project root (or `ROADMAP_DATA_DIR`).

### First-time VM note

Ubuntu images may lack `python3-venv`. If `python3 -m venv .venv` fails, run once:

```bash
sudo apt-get install -y python3.12-venv
```

Node.js 20+ is required only to build the React UI (`web/frontend/`). The built assets go to `web/static/dist/` (gitignored).

### Running the web admin (production-like)

From project root, with venv activated:

```bash
export ROADMAP_ADMIN_TOKEN=dev-admin
python -m uvicorn web.app:app --host 127.0.0.1 --port 8080
```

Open: `http://127.0.0.1:8080/?token=dev-admin`

If `web/static/dist/index.html` is missing, build first:

```bash
cd web/frontend && npm install && npm run build
```

### Frontend hot reload (optional)

Terminal 1 — API on 8080 (same as above).

Terminal 2:

```bash
cd web/frontend && npm run dev
```

Open `http://127.0.0.1:5173` (Vite proxies `/api` and `/health` to port 8080).

### CLI renderer (no server)

```bash
python render.py
```

### Lint / tests

This repo has no configured lint scripts, test runner, or pre-commit hooks. Verification is manual: `GET /health`, load the UI, and optionally `python render.py`.

### Environment variables

| Variable | Purpose |
|----------|---------|
| `ROADMAP_ADMIN_TOKEN` | Required for all `/api/*` routes (Bearer header or `?token=` query param) |
| `ROADMAP_DATA_DIR` | Override data directory (default: project root) |

### Gotchas

- **`web/static/dist/` is gitignored.** After `git pull`, run `npm run build` in `web/frontend/` unless someone force-committed a pre-built dist.
- **Port 8080 conflicts:** Stop any existing uvicorn process before starting a new one.
- **Windows vs Linux:** `start-admin.ps1` is the Windows launcher; on Linux use the uvicorn command above.
- **Read-only health check:** `GET /health` does not require auth (used for load balancer probes).

See [README.md](README.md) and [docs/WORK-SETUP.md](docs/WORK-SETUP.md) for full setup details.

---
> Source: [wyattjank/Roadmap-Render-Project](https://github.com/wyattjank/Roadmap-Render-Project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
