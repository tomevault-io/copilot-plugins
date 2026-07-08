## ai-research-agent

> An autonomous multi-agent pipeline that generates complete software projects from

# Project: Auto Dev Company – Autonomous Multi-Agent Software Builder

An autonomous multi-agent pipeline that generates complete software projects from
a natural language idea. Uses NVIDIA NIM (primary) or Groq (fallback) as the LLM
backend, with 14 specialized agents and a self-learning loop.

## Tech Stack
- Runtime: Python 3.12+
- Framework: FastAPI (async)
- LLM: NVIDIA NIM API (OpenAI-compatible), Groq fallback
- Database: Neon PostgreSQL (async via SQLAlchemy + asyncpg)
- Queue: Celery + Redis
- Tracing: Custom pipeline tracer with artifact export
- Dashboard: Vanilla HTML/CSS/JS served by FastAPI

## Build & Verify
```
pip install -e ".[dev]"
uvicorn app.main:app --reload       # API + dashboard
python run_pipeline.py              # standalone pipeline run
python run_pipeline.py --provider groq  # use Groq instead of NIM
pytest tests/ -v                    # run tests
```

## Folder Structure
```
app/
  agents/          # 14 pipeline agents (one file per agent)
  api/             # FastAPI routers (projects, agents, pipeline, dashboard)
  dashboard/       # HTML/CSS/JS dashboard served as static files
  models/          # SQLAlchemy ORM models
  orchestrator/    # Pipeline graph, state machine, review gate, Celery tasks
  sandbox/         # Docker sandbox manager and executor
  schemas/         # Pydantic request/response schemas
  tools/           # LangChain tools (linter, AST analyzer, etc.)
  tracing/         # Agent thinking trace system
  config.py        # Pydantic Settings from .env
  database.py      # Async SQLAlchemy engine
  logging.py       # Structlog JSON logging
  main.py          # FastAPI application entry point
  token_tracker.py # LLM token usage tracking
output/            # Generated project files + traces
tests/
```

## Code Style Rules
- Functions: single responsibility, aim for under 30 lines.
- Names: descriptive, no abbreviations (`resolve_import_path`, not `res_imp_pth`).
- Every exported function/class gets a one-line doc comment: what it does, not how.
- Comment only non-obvious logic inline — don't narrate obvious code.
- One file = one responsibility. Each agent gets its own file.
- Files stay under ~150 lines. If a file grows past that, split it.
- Before writing a new agent, check `app/agents/` for existing logic.

## Anti-Patterns (explicitly forbidden)
- No catch-all files (`utils.py`, `helpers.py`, `misc.py`) that accumulate unrelated logic.
- No commented-out dead code — delete it, git history keeps it.
- No vague names (`data`, `temp`, `handle_stuff`, `do_thing`).
- No inserting rows one at a time into Postgres — always batch.
- No hardcoded API keys — use .env and `app.config.settings`.
- No duplicating `_safe()` helpers — use the shared one.

## Database Rules
- Never insert into a high-volume table row-by-row — always batch.
- Worker pool size = `os.cpus().length`. Do not oversubscribe.

## Workflow Expectations
- Run `ruff check` before declaring any task done.
- When adding a new agent, extend the existing BaseAgent pattern.
- Ask before altering the DB schema in a way that isn't additive.

---
> Source: [MouleeswaranR/ai_research_agent](https://github.com/MouleeswaranR/ai_research_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
