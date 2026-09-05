## levelang-mcp

> Stateless MCP server that exposes the Levelang translation API to AI assistants (Claude, Cursor, etc.). Translates MCP tool calls into HTTP requests to the backend.

# Levelang MCP Server

Stateless MCP server that exposes the Levelang translation API to AI assistants (Claude, Cursor, etc.). Translates MCP tool calls into HTTP requests to the backend.

## Tech Stack

- **Framework**: Python MCP SDK
- **Package manager**: uv
- **Transport**: stdio (local) or streamable-http (deployed)
- **Deployment**: Fly.io, region `cdg`, port 8080

## Architecture

No database, no shared state, no code shared with the backend.

```
src/levelang_mcp/
├── __main__.py       # Entrypoint (python -m levelang_mcp)
├── server.py         # MCP tools and resources definitions
├── auth.py           # API-key auth middleware for HTTP transport
├── client.py         # Async HTTP client for the Levelang backend API
├── config.py         # Environment variable loading
├── formatting.py     # API response → human-readable text for LLMs
└── logging_config.py # Logging setup
```

## Key Coupling Points

- `client.py` — Makes HTTP calls to the backend. Must stay in sync with backend API endpoints and response schemas.
- `formatting.py` — Formats backend JSON into human-readable text for the LLM. Must stay in sync with the backend translate response schema.
- `server.py` — Defines MCP tools (`translate`, `translate_compare`, `list_languages`) and resources (`levelang://languages`).

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `LEVELANG_API_BASE_URL` | `http://localhost:8000/api/v1` | Backend URL |
| `LEVELANG_API_KEY` | — | Service key (`sk_*`) for backend auth |
| `MCP_TRANSPORT` | `stdio` | `stdio` or `streamable-http` |
| `MCP_PORT` | `8463` | Port for HTTP transport |
| `MCP_API_KEYS` | — | Comma-separated client auth keys for HTTP |

## Commands

```bash
uv sync                              # Install dependencies
uv run pytest tests/ -v              # Run tests
git config core.hooksPath .githooks  # Enable pre-commit/pre-push hooks
```

---
> Source: [beverage/levelang-mcp](https://github.com/beverage/levelang-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
