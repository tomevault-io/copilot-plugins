## wakalat-ai-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies (uses uv + pyproject.toml)
uv sync

# Run STDIO MCP server (for Claude Desktop / MCP Inspector)
uv run main.py

# Run HTTP/SSE MCP server (for web clients)
uv run http_main.py

# Test with MCP Inspector
npx @modelcontextprotocol/inspector uv run main.py
DANGEROUSLY_OMIT_AUTH=true npx @modelcontextprotocol/inspector uv run main.py

# Run tests
uv run pytest tests/
uv run pytest tests/test_tools.py::test_name  # single test

# Load sample data into vector DB
uv run python scripts/load_sample_data.py

# Lint / format / typecheck
uv run black src/
uv run ruff check src/
uv run mypy src/
```

## Architecture

There are **two server entry points** that share the same tool implementations:

- **`src/server.py`** — STDIO MCP server using `mcp.server.Server` + `stdio_server`. Used for Claude Desktop and MCP Inspector.
- **`src/http_server.py`** — HTTP/SSE server using FastAPI. Exposes `/sse` (MCP over SSE), `/tools` (REST list), `/tools/execute` (REST call), and `/ws` (WebSocket). The SSE endpoint delegates to the same `mcp_server` instance from `src/server.py`.

**Critical pattern**: Both `src/server.py` and `src/http_server.py` maintain their own copy of tool registration (`list_tools` / `get_available_tools`) and dispatch (`call_tool` / `execute_tool`). **When adding a new tool, you must update both files.**

### Tool implementation pattern

All tools live in `src/tools/` and follow the same pattern:
- Async function returning a JSON string
- All errors caught and returned as `{"error": "..."}` JSON
- Imports from `src.config.settings` for API keys / feature flags

### Currently implemented tools

| Tool | File | Status |
|---|---|---|
| `search_precedents` | `tools/precedent_search.py` | Live — uses Firecrawl REST API to scrape IndianKanoon |
| `deep_research` | `tools/deep_research.py` | Live — 4-step pipeline: Gemini query decomposition → Firecrawl parallel scrape → Gemini per-page extraction → Gemini synthesis |
| `find_case_laws` | `tools/case_law_finder.py` | Stub |
| `analyze_document` | `tools/document_analyzer.py` | Stub |
| `legal_research` | `tools/legal_research.py` | Stub (placeholder JSON) |
| `draft_legal_notice` | `tools/document_drafter.py` | Stub |
| `check_limitation` | `tools/limitation_checker.py` | Stub |

### `deep_research` pipeline detail

`src/tools/deep_research.py` uses `google-genai` SDK (`client.models.generate_content`) with `response_mime_type="application/json"` + `response_schema` dict for structured output at every step. Firecrawl is called via `FirecrawlApp` SDK (not raw HTTP like `precedent_search`). All Gemini calls are wrapped in `asyncio.get_event_loop().run_in_executor(None, lambda: ...)` since the SDK is sync.

### Config (`src/config.py`)

`Settings` is a `pydantic_settings.BaseSettings` loaded from `.env`. Key fields:
- `gemini_api_key` / `gemini_model` — used by `deep_research`
- `firecrawl_api_key` — used by both `precedent_search` and `deep_research`
- `enable_*` feature flags — gate tool registration in both servers

### Vector DB (`src/services/vector_db.py`)

Singleton `VectorDBService` backed by ChromaDB. Embeddings use `sentence-transformers` (`all-MiniLM-L6-v2`) when `USE_TRANSFORMERS=true` in env; otherwise falls back to a deterministic hash-based embedding. Currently used by `precedent_search` implicitly — not yet wired into `deep_research`.

### Dependencies

Managed via `pyproject.toml` (hatchling build, uv toolchain). Do **not** add dependencies to `requirements.txt` — it is legacy and not the source of truth.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BE-project-vesit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BE-project-vesit)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
