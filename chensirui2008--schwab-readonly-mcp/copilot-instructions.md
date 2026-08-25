## schwab-readonly-mcp

> This is a local, stdio-based Model Context Protocol (MCP) server for **read-only** Charles Schwab market and account data. Application code lives in `src/schwab_readonly_mcp/`: `server.py` exposes MCP tools, `client.py` calls Schwab APIs, `auth.py` manages OAuth, `config.py` validates settings, and `cli.py` provides the command-line entry point. Tests live in `tests/` and should mirror the module under test. Project metadata and tool configuration are in `pyproject.toml`; `uv.lock` pins the development environment.

# Repository Guidelines

## Project Structure & Architecture

This is a local, stdio-based Model Context Protocol (MCP) server for **read-only** Charles Schwab market and account data. Application code lives in `src/schwab_readonly_mcp/`: `server.py` exposes MCP tools, `client.py` calls Schwab APIs, `auth.py` manages OAuth, `config.py` validates settings, and `cli.py` provides the command-line entry point. Tests live in `tests/` and should mirror the module under test. Project metadata and tool configuration are in `pyproject.toml`; `uv.lock` pins the development environment.

## Setup, Test, and Development Commands

Use Python 3.11+ and `uv`:

```bash
uv sync                              # install locked dependencies
uv run pytest                        # run all tests
uv run ruff check .                  # lint source and tests
uv run schwab-readonly-mcp server    # start the stdio MCP server
uv run schwab-readonly-mcp auth      # complete OAuth and store a local token
```

For a lightweight import check, run:

```bash
uv run python -c "from schwab_readonly_mcp.server import create_server; print(create_server)"
```

## Coding Style & Naming

Use Python with four-space indentation, type annotations for public interfaces, and a 100-character maximum line length (Ruff is authoritative). Use `snake_case` for functions, variables, modules, and test names; use `PascalCase` for classes and exceptions. Keep MCP tool names and API operations explicitly read-only. Do not catch exceptions merely to hide failures—raise clear, actionable errors at the boundary instead.

## Testing Guidelines

Write pytest tests named `test_<behavior>.py` and individual tests named `test_<expected_outcome>`. Cover successful parsing or API mapping as well as invalid configuration, missing OAuth fields, and API error paths. Use `monkeypatch` for environment variables and temporary paths; never depend on a real Schwab account, network call, or saved token in tests. Run both `uv run pytest` and `uv run ruff check .` before committing.

## Security, Commits, and Pull Requests

Never commit `SCHWAB_CLIENT_SECRET`, OAuth redirects, tokens, account data, or `.env` files. Tokens belong in the owner-only default path `~/.config/schwab-readonly-mcp/token.json`.

Use concise Conventional Commit messages, for example `feat: add read-only quote tool` or `docs: add contributor guide`. Keep each commit focused. Pull requests should explain the user-visible change, list validation commands run, link relevant issues, and call out any new environment variables or Schwab API permissions. Include sample MCP output only when it is redacted.

---
> Source: [chensirui2008/schwab-readonly-mcp](https://github.com/chensirui2008/schwab-readonly-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
