## mcp-agent-forge

> This is MCP Agent Forge -- an MCP server that dispatches AI coding agents.

# CLAUDE.md

This is MCP Agent Forge -- an MCP server that dispatches AI coding agents.

## Structure
- `server.py` -- FastMCP server. All tools and agent runners in one file.
- `README.md` -- User-facing docs.
- `pyproject.toml` -- Package config.

## Rules
- Keep server.py as a single file. No splitting into modules.
- All agent runners are async functions named `_run_{agent}`.
- Output is always JSON via `json.dumps()`.
- No external dependencies beyond `fastmcp`.
- ASCII only in source. No emoji, no curly quotes.

---
> Source: [Cloud-Eye-Prime/mcp-agent-forge](https://github.com/Cloud-Eye-Prime/mcp-agent-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
