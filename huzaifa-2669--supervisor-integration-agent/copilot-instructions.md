## supervisor-integration-agent

> This repo hosts a FastAPI + React demo where a supervisor LLM orchestrates worker agents. Use this guide to contribute consistently and safely.

# Repository Guidelines

This repo hosts a FastAPI + React demo where a supervisor LLM orchestrates worker agents. Use this guide to contribute consistently and safely.

## Project Structure & Module Organization
- `main.py`: Entrypoint that imports the FastAPI app.
- `app/`: Package with modules:
  - `server.py`: FastAPI wiring and routes.
  - `registry.py`: Agent definitions and lookup helpers.
  - `planner.py`: LLM planner with fallback plan.
  - `agent_caller.py`: Handshake builder and HTTP agent calls (returns structured errors if endpoints/httpx fail).
  - `executor.py`: Plan execution and input resolution.
  - `answer.py`: Final answer synthesis (LLM + fallback).
  - `models.py`: Shared Pydantic schemas.
  - `web.py`: React UI served from `/`.

## Build, Test, and Development Commands
- `uvicorn main:app --reload` — run the dev server at http://localhost:8000.
- `python main.py` — convenience entrypoint; proxies to uvicorn with reload.
- `pytest` (add later) — preferred for automated tests; use `httpx.AsyncClient` + FastAPI `TestClient` for API tests.

## Coding Style & Naming Conventions
- Python 3.10+; prefer type hints and Pydantic models for all contracts.
- Indent with 4 spaces; keep lines reasonably short and comments succinct.
- Function names: `snake_case`; classes: `PascalCase`; registry agent names: `snake_case_agent`.
- UI uses React (CDN + Babel). Keep components small; co-locate UI tweaks in `web.py`.

## Testing Guidelines
- Add unit tests for planner fallbacks, registry lookups, and `/api/query` happy-path/error cases.
- When stubbing agents, assert handshake shape (request_id, status, output/error).
- Include at least one test that exercises the debug payload structure (`used_agents`, `intermediate_results`).

## Commit & Pull Request Guidelines
- Commit messages: imperative mood (e.g., "Add planner fallback when OpenAI missing").
- PRs: describe scope, risks, and how you tested; attach screenshots of the UI if changed.
- Keep PRs small and scoped; call out breaking changes and required env vars (e.g., `OPENAI_API_KEY`).

## Security & Configuration
- Secrets: never commit; rely on environment variables (`OPENAI_API_KEY`).
- Networked agents: validate endpoints before enabling real HTTP calls. Timeouts are set per agent; keep them conservative.
- If adding persistent storage, document schema and migration steps before merging.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Huzaifa-2669)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Huzaifa-2669)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
