## mcp-example

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP (Model Context Protocol) examples using the FastMCP Python library. The project demonstrates building MCP servers with tools and middleware-based authentication, plus a client for calling those tools remotely.

## Setup & Commands

This project uses `uv` for Python package management with Python 3.12.

```bash
# Install dependencies
uv sync

# Run the MCP server
uv run -m mcp_examples.server

# Run the MCP client
uv run -m mcp_examples.cli

# Run tests
uv run pytest
```

## Architecture

The project is organized as a `mcp_examples` Python package:

- **mcp_examples/database.py** — `DatabaseManager` class for SQLite-backed API key storage and validation.
- **mcp_examples/middleware.py** — `ApiKeyMiddleware` class that checks for an `X-API-Key` header on incoming HTTP requests.
- **mcp_examples/base.py** — `AuthenticatedMCPServer` abstract base class with API key auth infrastructure (database, middleware). Subclass and implement `_register_tools()` to create new servers.
- **mcp_examples/server.py** — `GreetMCPServer` derived class demonstrating tool registration. Exposes module-level `server` and `mcp` instances.
- **mcp_examples/client.py** — `MCPClient` class that connects to a remote MCP endpoint and calls tools.
- **mcp_examples/cli.py** — `MCPClientApp` CLI application and `main()` entry point.
- **mcp_examples/__init__.py** — Re-exports all public classes.
- **auth.py** — Standalone snippet of the `ApiKeyMiddleware` class (no imports; reference/example code, not directly runnable).

## Key Dependencies

- `fastmcp>=2.14.5` — The core library for both server and client. Provides `FastMCP`, `Client`, `Middleware`, `ToolError`, and `get_http_headers`.

---
> Source: [dgwartney/mcp-example](https://github.com/dgwartney/mcp-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
