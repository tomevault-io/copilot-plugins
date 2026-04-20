## google-scholar-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Install dependencies (use uv, not pip)
uv pip install -e ".[dev]"

# Check syntax for all source files
python -m py_compile src/main.py src/config.py src/clients/*.py src/models/*.py src/tools/*.py

# Run tests
pytest tests/ -v

# Run MCP server locally (stdio transport, requires .env with API keys)
python -m src.main
```

## Architecture

This is a **FastMCP-based MCP server** that provides Google Scholar search and Open Access full-text retrieval.

### Key Patterns

1. **Lifespan Context Manager** (`src/main.py`): Clients are initialized in the `lifespan` async context manager, not via decorators. Global references (`_serpapi_client`, `_core_client`) are set during startup and cleaned up on shutdown.

2. **Tool Registration**: Tools are registered via `register_*_tools(mcp, get_client_fn)` functions that receive getter functions for lazy client access.

3. **Async HTTP Clients**: Both `SerpAPIClient` and `CoreAPIClient` use `httpx.AsyncClient` with:
   - `follow_redirects=True` (required for CORE API)
   - Retry with exponential backoff on transient errors (429, 500, 502, 503, 504)
   - Unified error hierarchy: `APIError`, `RateLimitError`, `AuthenticationError`
   - Proper cleanup via `close()` method

### API Specifics

- **SerpAPI**: Uses `engine=google_scholar` for articles, `cites` param for citations, `cluster` param for versions
- **CORE API**: Base URL `https://api.core.ac.uk/v3`, requires Bearer token auth, endpoints need trailing slashes

## Environment Variables

- `SERPAPI_API_KEY` (required) - from serpapi.com
- `CORE_API_KEY` (optional) - from core.ac.uk for full-text access

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alimov-andrey) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
