## agent-memory

> MCP (Model Context Protocol) service backed by SQLite, with a co-located Starlette web dashboard.

# Agent Memory Server

MCP (Model Context Protocol) service backed by SQLite, with a co-located Starlette web dashboard.

## CRITICAL REQUIREMENT

**ALL responses and plans MUST be in English only.** Even if the user writes in another language (e.g., Russian), respond and create execution plans in simple English with IT terminology. Do not switch to the user's language.

## MANDATORY WORKFLOW

When receiving any new task from the user in plan mode, first read the value of the key `new-dev-task` from memory and follow the instructions provided there.

## Tech Stack

- Python 3.12+
- `mcp` (FastMCP) for MCP tools
- `starlette` + `jinja2` + `uvicorn` for HTTP dashboard
- SQLite for key-value storage

## Entry Points

- `py -m agent_memory` — stdio transport for MCP clients
- `py src/agent_memory/main.py` — HTTP server on `127.0.0.1:8765`
  - Dashboard: `/dashboard`
  - SSE MCP endpoint: `/mcp`

## Key Files

| Path | Purpose |
|------|---------|
| `src/agent_memory/server.py` | FastMCP tools (`put`, `get`) + Starlette routes |
| `src/agent_memory/storage.py` | SQLite wrapper: `init_db`, `put`, `get`, `list_keys`, `delete` |
| `src/agent_memory/main.py` | Uvicorn runner |
| `src/agent_memory/templates/dashboard.html` | Dashboard UI |
| `src/agent_memory/static/style.css` | Dashboard styles |
| `data/memory.db` | SQLite database |

## Testing

```bash
py -m pytest tests/ -v
```

---
> Source: [fraken-baken/agent-memory](https://github.com/fraken-baken/agent-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
