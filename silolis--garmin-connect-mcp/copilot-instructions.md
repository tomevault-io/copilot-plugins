## garmin-connect-mcp

> Always use `uv` to run Python code to ensure correct dependencies:

# Claude Code Project Instructions

## Running Code

Always use `uv` to run Python code to ensure correct dependencies:

```bash
uv run python garmin_connect_mcp.py
uv run pytest
```

## Project Overview

This is an MCP (Model Context Protocol) server for Garmin Connect that provides tools to access fitness and health data.

### Key Files

- `garmin_connect_mcp.py` - Main MCP server implementation
- `pyproject.toml` - Project dependencies managed by uv

### Environment Variables

Required for authentication:
- `GARMIN_EMAIL` - Garmin Connect email
- `GARMIN_PASSWORD` - Garmin Connect password

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/silolis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/silolis)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
