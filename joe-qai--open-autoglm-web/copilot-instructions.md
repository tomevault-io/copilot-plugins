## open-autoglm-web

> Multi-platform mobile agent testing platform. VLM-driven phone automation via ADB (Android), HDC (HarmonyOS), XCTest (iOS).

# AGENTS.md — Open-AutoGLM (LOCKIN Agent Platform)

Multi-platform mobile agent testing platform. VLM-driven phone automation via ADB (Android), HDC (HarmonyOS), XCTest (iOS).

## Quick start

```bash
# Backend (port 8005)
cd backend && pip install -r requirements.txt && python run.py

# Frontend (port 3000, proxies API to :8005)
cd frontend && npm install && npm run dev

# CLI mode
python main.py --base-url <URL> --model <NAME> "task description"

# Batch start
start_all.bat
# Kill ports 8005 + 3000
kill_ports.bat
```

## Testing

```bash
# Backend only (asyncio_mode=auto)
cd backend && pytest

# Manual integration test
cd backend && python test_agent_engine.py
```

## Key paths

| Purpose | Path |
|---|---|
| CLI entry | `main.py` (imports `backend.app.core.*`) |
| FastAPI app | `backend/app/main.py` (port 8005) |
| API routes | `backend/app/api/v1/` (11 routers) |
| Agent engine | `backend/app/core/agent/engine.py` — coordinates ReActLoop |
| Platform adapters | `backend/app/core/adapters/{android,ios,harmonyos}.py` |
| Agent layers | `backend/app/core/layers/{perception,decision,action,memory,verification}.py` |
| Device drivers | `backend/app/core/devices/{adb,hdc,xctest}/` |
| Services | `backend/app/services/` (task, device, script, etc.) |
| DB schema | `backend/app/db/database.py` — SQLite via aiosqlite (no migrations) |
| App package map | `backend/app/core/config/app_packages.py` (50+ apps) |
| Prompts | `backend/app/core/config/i18n.py` |
| Frontend root | `frontend/src/App.tsx` — React + Vite + Tailwind |
| Tests | `backend/tests/` |
| Env template | `backend/.env.example` |

## Backend architecture

```
AgentEngine
├─ ReActLoop (Observe → Think → Act → Reflect)
├─ Layers: Perception → Decision → Action → Memory → Verification → Replay
├─ Sub-agents: Manager, Executor, Reflector, Finder
├─ Adapter per platform: Android(ADB), iOS(WDA), HarmonyOS(HDC)
└─ DeviceModelAdapter → DeviceDriver → ADB/HDC/XCTest binary
```

## Environment variables (all prefixed `PHONE_AGENT_`)

| Key | Default | Note |
|---|---|---|
| `MODEL_API_URL` | `http://localhost:8000/v1` | OpenAI-compatible VLM |
| `MODEL_NAME` | `lockin-phone-9b` | Model served |
| `API_KEY` | `EMPTY` | |
| `API_HOST` / `API_PORT` | `0.0.0.0:8005` | FastAPI host:port |
| `DATABASE_URL` | `./app.db` | SQLite |

## Important conventions

- **Windows encoding**: prefix with `PYTHONIOENCODING=utf-8` if garbled output
- **Format**: `pseudo` (default) or `json` — affects action output format
- **Schema**: SQLite tables auto-created at startup; use `db.get_connection()` for raw SQL
- **Model API**: must be OpenAI-compatible; supports both local (vLLM/SGLang) and cloud (BigModel/ModelScope)
- **Lint**: Ruff in pre-commit; run manually: `ruff check backend/`
- **Pre-commit**: installed but may be skipped locally
- **App packages**: centralized in `app_packages.py` — add new app entries there

---
> Source: [joe-qai/Open-AutoGLM-Web](https://github.com/joe-qai/Open-AutoGLM-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
