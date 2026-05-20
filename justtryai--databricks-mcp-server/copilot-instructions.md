## project-structure

> Project Structure

# Databricks MCP Server Structure

## Source Code Organization
- `src/server/` - MCP server implementation (files matching `*_mcp_server.py` and `*.py`)
- `src/api/` - API client for Databricks services
- `src/core/` - Core functionality and utilities
- `src/cli/` - Command-line interface

## Testing
- `tests/` - Test files (matching `test_*.py`)
- Each file in `src/` should have a corresponding test file in `tests/`

## Additional Directories
- `examples/` - Example usage of the MCP server
- `scripts/` - Helper scripts for running the server and tests (`.ps1`, `.sh` files)

## Required Files
- README.md
- pyproject.toml
- .gitignore
- src/server/databricks_mcp_server.py

---
> Source: [JustTryAI/databricks-mcp-server](https://github.com/JustTryAI/databricks-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
