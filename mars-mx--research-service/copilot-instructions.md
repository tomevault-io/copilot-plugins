## research-service

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
uv sync

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_engine.py

# Run a single test by name
uv run pytest tests/test_engine.py::test_resolve_params_quick

# Run the app locally (requires .env with API keys + Redis running)
uv run uvicorn src.main:app --host 0.0.0.0 --port 8000

# Lint (same as CI)
uv run python -m py_compile src/main.py

# Local dev with Docker (app + Redis)
docker-compose up
```

## Architecture

HTTP service that performs autonomous multi-step web research. A single-process FastAPI app backed by Redis for result caching.

**Research pipeline:** plan → search → scrape → compress → write report

Each stage is a discrete module in `src/research/`:
- `engine.py` — Orchestrator. `ResearchEngine.run()` drives the full pipeline. Recursion controlled by depth/breadth params. Emits SSE events via `EventCallback` for streaming.
- `depth.py` — Preset tiers (quick/standard/deep) mapping to (depth, breadth, min_words). `resolve_params()` in engine.py resolves tier vs custom overrides.
- `search.py` — Tavily async wrapper
- `scrape/` — Pluggable loader registry (strategy pattern). `ScraperRegistry` matches URL domains to `PageLoader` implementations. Currently only `FirecrawlLoader`.
- `compress.py` — Embeddings (OpenAI or Google Gemini) + numpy cosine similarity to rank passages by relevance
- `prompts.py` — Prompt templates with format helpers for plan/follow-up/report stages
- `events.py` — `EventCallback` type alias and `emit_event()` helper
- `tasks.py` — Background task runner, callback URL validation (SSRF protection), retry logic

**API layer** (`src/api/`):
- `routes.py` — Thin endpoint handlers. `POST /research` dispatches to stream or background mode.
- `service.py` — Service layer between routes and engine. Manages SSE queue, background task creation, and cache writes.
- `schemas.py` — Pydantic models: `ResearchRequest`, `ResearchResult`, `ResearchSource`, `ResearchMetadata`

**Two execution modes:**
1. **Streaming** — SSE via `EventSourceResponse`. Research runs in an asyncio task feeding an `asyncio.Queue`; the generator yields events. On client disconnect, the task continues to completion (result always cached).
2. **Background** — Returns 202 immediately. Research runs via `asyncio.create_task`. Result cached in Redis, then POSTs to `callback_url`.

**Dependency injection:** `app.state` holds `settings`, `cache`, and `engine` (set in `lifespan`). Route handlers pull them via FastAPI `Depends()` on private `_get_*` functions.

## Testing Patterns

- **pytest-asyncio in strict mode** (default): use `@pytest.mark.asyncio` on each async test. Use `@pytest_asyncio.fixture` (not `@pytest.fixture`) for async fixtures.
- **fakeredis**: `FakeRedis(decode_responses=True)` for in-memory Redis in tests (see `conftest.py`).
- **Mocking external services**: Tests mock `Agent`, `search`, `scrape`, and `compress_context` via `unittest.mock.patch`. Engine tests create a `MagicMock` Settings object rather than loading real env vars.
- Tests mix sync and async in the same file (e.g., `test_engine.py` has sync `resolve_params` tests and async engine tests) — do NOT use module-level `pytestmark = pytest.mark.asyncio` in mixed files.

## Key Dependencies

- **PydanticAI 1.56.0** — `Agent(model, output_type=T)` → `result.output` (validated T), `result.usage()` (token counts with `.input_tokens`, `.output_tokens`, `.requests`)
- **uv** — Package manager (lockfile: `uv.lock`)
- **Pydantic Settings** — Config loaded from env vars / `.env` file via `src/config.py`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mars-mx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
