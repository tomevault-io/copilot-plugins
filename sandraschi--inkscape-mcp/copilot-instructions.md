## inkscape-mcp

> MCP server for Inkscape-backed SVG and vector operations (FastMCP 3.2.0+)

# inkscape-mcp — Claude Code Guide

## Overview
MCP server for Inkscape-backed SVG and vector operations (FastMCP 3.2.0+)

## Entry Points
- `uv run inkscape-mcp` → `inkscape_mcp.main:main`

## Standards
- FastMCP 3.2+ portmanteau tool pattern — tools use `operation` enum param
- Responses: structured dicts with `success`, `message`, domain-specific fields
- Dual transport: stdio (Claude Desktop) + HTTP (`MCP_TRANSPORT=http`)
- See [mcp-central-docs](https://github.com/sandraschi/mcp-central-docs) for fleet-wide coding standards

## Key Files
- `README.md` — full documentation
- `pyproject.toml` — build config and entry points
- `AGENTS.md` — OpenAI Codex agent context (if present)

---
> Source: [sandraschi/inkscape-mcp](https://github.com/sandraschi/inkscape-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
