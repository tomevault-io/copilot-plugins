## mcp-pdb

> This project is a debugging tool that integrates PDB (Python Debugger) with the MCP (Multi-modal Chat Protocol) framework. It allows interactive debugging of Python code through a chat interface.

# mcp-pdb Development Guide

## Project Overview
This project is a debugging tool that integrates PDB (Python Debugger) with the MCP (Multi-modal Chat Protocol) framework. It allows interactive debugging of Python code through a chat interface.

## Build Commands
- Run the server: `uv run mcp-pdb`
- Lint: `uv run ruff check`
- Type check: `uv run pyright`

## Development Environment

### Package Management
- This project uses `uv` for package management instead of pip/pip-tools
- Commands should use the `uv run` prefix for running Python tools
- Detection of uv environment is done via `uv tree` command

### Debugging
- When debugging with `uv`, use `uv run pdb <file_path>` instead of `python -m pdb`
- When testing with pytest, use `uv run pytest --pdb <file_path>`

### Project Structure
- Main functionality is in `src/mcp_pdb/main.py`
- The code provides MCP tools for:
  - Starting/stopping debug sessions
  - Setting/managing breakpoints
  - Sending commands to PDB
  - Examining variables
  - Managing debug status

### Development Notes
- The project uses the FastMCP framework for building MCP tools
- Detection of uv environments is done by running `uv tree` in the project directory
- Breakpoints are tracked across debug sessions

---
> Source: [samefarrar/mcp-pdb](https://github.com/samefarrar/mcp-pdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
