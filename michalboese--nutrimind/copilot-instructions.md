## nutrimind

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies (venv is already present)
source venv/Scripts/activate
pip install -r requirements.txt

# Run FastAPI backend
uvicorn app.main:app --reload          # http://localhost:8000
# Interactive API docs: http://localhost:8000/docs

# Run Gradio UI (standalone, no FastAPI needed)
python ui/gradio_app.py                # http://localhost:7860

# Run all tests
pytest

# Run a single test
pytest tests/test_api.py::test_create_meal_success
```

Ollama must be running locally for end-to-end usage:
```bash
ollama run llama3.2
```

## Architecture

Calorie tracking app with an AI agent that estimates nutritional values from natural-language meal descriptions.

### Component map

```
ui/gradio_app.py          3-tab web UI (add meal / history / daily summary)
        │
        └── imports directly ──► app/agent.py      async analyze_meal() → Ollama
                                  app/database.py   SQLite via run_in_executor
                                  app/models.py     Pydantic v2 models

app/main.py               FastAPI app (independent entry point, same imports)
```

The UI and the API are **two separate entry points** that share the same `app/` layer. They do not communicate with each other over HTTP.

### Key design decisions

- **`app/agent.py`** — sends meal description to `POST /api/chat` on Ollama (`llama3.2`). System prompt enforces JSON-only output. A regex strips markdown code fences in case the model wraps the JSON anyway.
- **`app/database.py`** — uses stdlib `sqlite3` wrapped in `asyncio.run_in_executor` to avoid blocking FastAPI's event loop. DB file is `calorie_agent.db` in the project root (gitignored).
- **`app/models.py`** — three Pydantic v2 models: `MealRequest` (input), `MealRecord` (full DB row), `DailySummary` (aggregated totals).
- **`pytest.ini`** sets `asyncio_mode = auto` so async tests don't need `@pytest.mark.asyncio`. Tests patch `DB_PATH` via `unittest.mock.patch` to isolate each test in a `tmp_path` SQLite file.

### FastAPI endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/meals` | Analyze + save a meal, returns `MealRecord` |
| `GET` | `/meals` | List all meals (`?date=YYYY-MM-DD` to filter) |
| `GET` | `/meals/{id}` | Single meal by id |
| `GET` | `/summary` | Aggregated totals for a day (`?date=`, default today) |
| `GET` | `/health` | Health check |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michalboese) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
