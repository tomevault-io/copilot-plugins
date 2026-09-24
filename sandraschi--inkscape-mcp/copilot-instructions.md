## inkscape-mcp

> MCP server for Inkscape-backed SVG and vector operations (FastMCP 3.2.0+)

# inkscape-mcp — Agent Guide

## Overview
MCP server for Inkscape-backed SVG and vector operations (FastMCP 3.2.0+)

## Entry Points
- `uv run inkscape-mcp` → `inkscape_mcp.main:main`

## Standalone Desktop Installer Required
Inkscape MUST be installed via the **classic desktop installer** from https://inkscape.org/release/
The Microsoft Store version is **sandboxed** and its `inkscape.exe` cannot run CLI commands — the server will fail with "Access Denied" when calling `inkscape --version` or any batch operation.

Verify: `& "C:\Program Files\Inkscape\bin\inkscape.exe" --version`
Custom path: set `INKSCAPE_PATH` env var

## Standards
- FastMCP 3.2+ portmanteau tool pattern — tools use `operation` enum param
- Responses: structured dicts with `success`, `message`, domain-specific fields
- Dual transport: stdio (Claude Desktop) + HTTP (`MCP_TRANSPORT=http`)
- See [mcp-central-docs](https://github.com/sandraschi/mcp-central-docs) for fleet-wide coding standards

## Key Files
- `README.md` — full documentation
- `pyproject.toml` — build config and entry points
- `CLAUDE.md` — Claude Code context (if present)

Install docs: follow mcp-central-docs/standards/AGENT_INSTALL_REFERENCE.md

---
> Source: [sandraschi/inkscape-mcp](https://github.com/sandraschi/inkscape-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
