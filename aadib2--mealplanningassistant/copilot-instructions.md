## mealplanningassistant

> - This repo is a single Python project (no monorepo/tooling layers) with two runnable surfaces:

# Meal Planning Assistant Project AGENTS.md

## Project Summary
- This repo is a single Python project (no monorepo/tooling layers) with two runnable surfaces:
  - FastAPI backend: `backend/app/api.py`
  - Streamlit UI: `app.py`
- Session chat flow is: `app.py` -> `/chat` route in `backend/app/api.py` -> `SessionChatHandler` (`backend/app/session_chat_handler.py`) -> Pinecone retrieval via `rag/query.py` + Anthropic response.
- Recipe ingestion flow is: `/ingest` route -> `Ingester` (`backend/app/ingester.py`) -> Spoonacular fetch -> normalize/validate/transform (`rag/normalize.py`, `rag/validate.py`, `rag/transform.py`) -> Pinecone upsert.

##  Project Structure
- `app.py`: Streamlit frontend and API client for `/chat`, `/preferences`, `/session-preferences`, and `/ingest`.
- `backend/app/`: FastAPI app and runtime services.
  - `backend/app/api.py`: main API routes and request/response schemas.
  - `backend/app/session_chat_handler.py`: chat orchestration, retrieval calls, Anthropic invocation.
  - `backend/app/ingester.py`: Spoonacular fetch + Pinecone indexing pipeline.
  - `backend/app/redis_session_manager.py`: Redis-backed chat history persistence.
  - `backend/app/user_memory_store.py`: JSON-backed user/session preference store (`data/user_memory.json`).
- `rag/`: retrieval and ingestion helpers.
  - `rag/query.py`: Pinecone retrieval used by backend chat.
  - `rag/normalize.py`, `rag/validate.py`, `rag/transform.py`: recipe normalization/validation/document building.
  - `rag/ingestion.py`, `rag/index.py`: standalone scripts for fetching and indexing recipes.
- `data/`: local data and constants.
  - `data/spoonacular_data_options.py`: allowed filter values used by UI and backend validation.
  - `data/raw_recipes.json`: local cache/input for standalone indexing.
  - `data/user_memory.json`: runtime-mutated user/session memory file.
- `agents/`: legacy LangChain agent/tool experiments; not part of the active FastAPI + Streamlit runtime path.

## Required local services and env
- Backend startup imports `SessionChatHandler` and `Ingester` at module load, so missing env vars fail fast before the server starts.
- Required env vars for normal backend usage: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `PINECONE_API_KEY`, `SPOONACULAR_API_KEY` (plus optional `PINECONE_INDEX_NAME`, `PINECONE_CORE_NAMESPACE`, `EMBEDDING_MODEL`).
- Redis must be running at `redis://localhost:6379` unless code is changed; chat history persistence depends on it (`backend/app/redis_session_manager.py`).

## Commands agents should use
- Install deps: `uv sync`
- Run backend from repo root: `uv run uvicorn backend.app.api:app --reload`
- Run Streamlit UI: `uv run streamlit run app.py`
- Legacy/standalone RAG scripts:
  - Fetch/cache raw recipes: `uv run python rag/ingestion.py`
  - Build Pinecone index from `data/raw_recipes.json`: `uv run python rag/index.py`

## Verification reality (important)
- There is currently no configured automated test/lint/typecheck pipeline in-repo (no `tests/`, `pytest.ini`, `ruff` config, CI workflows, or pre-commit config).
- Practical verification is manual: start backend + Streamlit, then exercise `/chat`, `/preferences`, `/session-preferences`, and `/ingest`.

## Repo-specific gotchas
- `data/user_memory.json` is actively mutated at runtime by `UserMemoryStore`; avoid accidental commits of local session/preference data.
- `/ingest` validates filter values against `data/spoonacular_data_options.py`; invalid labels return 400 via Pydantic validators.
- `rag/verify.py` and `data/test_json.py` are ad-hoc debug scripts and are explicitly ignored in `.gitignore`; do not treat them as authoritative tests.

---
> Source: [aadib2/MealPlanningAssistant](https://github.com/aadib2/MealPlanningAssistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
