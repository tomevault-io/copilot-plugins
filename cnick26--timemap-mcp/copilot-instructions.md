## timemap-mcp

> MCP server wrapping TimeMap's Chinese metaphysics engine.

# timemap-mcp

MCP server wrapping TimeMap's Chinese metaphysics engine.

## Commands

| Command | Description |
|---------|-------------|
| `.venv/Scripts/python -m pytest -q` | Run all tests (740+) |
| `.venv/Scripts/python -m pytest tests/test_tools.py -v` | Run MCP tool tests only |
| `python scripts/sync_engine.py` | Sync engine from TimeMap repo |

## Architecture

- `timemap_mcp/engine/` — Pure calculation modules (synced from TimeMap)
- `timemap_mcp/server.py` — FastMCP tool definitions (8 tools)
- `timemap_mcp/_converters.py` — Human-readable input parsing
- `tests/engine/` — Engine unit tests (synced from TimeMap)
- `tests/test_tools.py` — MCP tool integration tests
- `scripts/sync_engine.py` — Engine sync script

## Engine Sync

Engine modules are copied from `../TimeMap/backend/app/engine/` with import path fixup.
Run `python scripts/sync_engine.py` after engine changes in TimeMap.

## Code Style

- Python 3.11+, snake_case, frozen dataclasses
- Engine modules are PURE functions — no I/O, no DB
- MCP tools return JSON strings (FastMCP handles protocol)


## Knowledge vault
Cunoaștere durabilă despre Nick (preferințe, principii, BaZi, oameni, decizii cross-project) e în `C:\Users\Nick\Documents\knowledge\`. Citește `insights/` cu Grep pe slug/topic când e relevant (ex: "nick birth data" → grep `birth-time`, "alex partnership" → grep `alex|newsletter`). NU scrie în vault fără permisiune Nick.

---
> Source: [cnick26/timemap-mcp](https://github.com/cnick26/timemap-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
