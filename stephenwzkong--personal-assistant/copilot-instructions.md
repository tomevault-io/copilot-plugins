## personal-assistant

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Permissions

- Reading files: allowed without asking for permission.
- Making any changes (edits, writes, deletions, running commands, etc.): always ask for permission first.

## Build and Run Commands

```bash
# Setup virtual environment
uv venv
source .venv/bin/activate

# Install dependencies
uv pip install -e .

# Run the Personal Assistant — FastAPI backend (port 8082)
cd src/assistant && uvicorn api.main:app --host 0.0.0.0 --port 8082 --reload

# Run the Personal Assistant — Vite frontend (port 5173, proxies /api → 8082)
cd src/assistant/frontend && pnpm install && pnpm dev

# Provision BigQuery tables for the assistant (one-time)
cd src/assistant && PYTHONPATH=. python schema/create_tables.py

# Run agents with Google ADK
cd tests && adk run agent.py
```

## Architecture Overview

This is a personal assistant project with two main components:

### 1. Personal Assistant (`src/assistant/`)
Multi-agent app with a Vite + React + shadcn frontend and a FastAPI backend. Single Cloud Run service on port 8082 (FastAPI serves `/api/*` and mounts the built `frontend/dist` at `/`). Vite dev server runs on 5173 with a `/api → 8082` proxy.

**Architecture**: Root `LlmAgent` orchestrator with domain routers (wellness, productivity, social) and standalone specialists (calendar, finance, trivial). Sub-agents use `AgentTool(calendar_agent)` to write calendar events. All data persisted to BigQuery (`personal_assistant` dataset).

**Sessions**: `FirestoreSessionService` (in `memory/firestore_session_service.py`) persists chat history to Firestore collections `adk_sessions` / `adk_user_state` / `adk_app_state`. Toggle with `PA_USE_FIRESTORE_SESSIONS=0` to fall back to in-memory.

**Memory**: `memory/service.py` stores user facts in BigQuery `memory_facts`. `inject_memory_bundle` before-agent callback injects a compact `<user_memory>` block into the orchestrator prompt via `{user_memory_block}`.

**Key files**:
- `agents/root_orchestrator.py` — root `LlmAgent` with domain routers + standalones as `sub_agents`
- `agents/shared/calendar_agent.py` — owns calendar CRUD; shared via `AgentTool`
- `runner.py` — ADK `Runner` singleton, session service selection, `run_agent` entrypoint
- `api/main.py` — FastAPI app; routes in `api/routes/{chat,calendar,session}.py`
- `db/bq_client.py` — BigQuery calendar query helper used by `api/routes/calendar.py`
- `memory/` — session service, fact store, bundle injection callback
- `frontend/src/app/components/` — React pages (Home, CalendarPage, DomainsPage, ChatInterface, DomainChat)
- `frontend/src/lib/api.ts` — typed fetch client for `/api/*`
- `schema/create_tables.py` — provisions BigQuery tables (run with `PYTHONPATH=.`)

### 2. Agent System (`tests/agent.py`)
Google ADK-based travel planning agent using `SequentialAgent` pattern. Chains three agents (research → itinerary → optimization) using `output_key` to pass state between steps. Uses `gemini-2.5-flash`.

## Key Integrations

- **Google Cloud Storage**: Bucket `personal_assistant_agent` — stores uploaded images
- **Gemini Vision**: `gemini-2.0-flash-preview` — analyzes food/workout images with structured JSON output
- **BigQuery**: Dataset `personal_assistant` — persists meal and workout records
- **Google ADK**: `SequentialAgent` for chaining LLM agents
- **LiteLLM**: Universal LLM interface (supports Ollama, OpenAI, Google)

## Environment Variables

Required in `.env`:
```
GOOGLE_API_KEY=your_key
GOOGLE_CLOUD_PROJECT=your_project_id
```

For service account auth: `GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json`

For Application Default Credentials (recommended for local dev):
```bash
gcloud auth application-default login
```

---
> Source: [stephenwzkong/personal_assistant](https://github.com/stephenwzkong/personal_assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
