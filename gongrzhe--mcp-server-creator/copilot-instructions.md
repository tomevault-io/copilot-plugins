## mcp-server-creator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Server Creator is a meta-MCP server that creates other MCP servers using FastMCP. It's a Python package that provides both programmatic APIs and MCP tools for dynamically generating complete MCP server configurations and Python code.

## Core Architecture

The project consists of:

- **Main Module**: `mcp_server_creator/mcp_server_creator.py` - Contains all MCP tools and core logic
- **Server Configurations**: In-memory storage using dataclasses (`ServerConfig`, `ToolConfig`, `ResourceConfig`)
- **Code Generation**: Template-based Python code generation for complete FastMCP servers
- **Entry Points**: Both CLI script and FastMCP server entry points in `pyproject.toml`

Key architectural patterns:
- Uses in-memory storage (`servers` dict) for managing server configurations during runtime
- Dataclass-based configuration management with automatic imports tracking
- Template-based code generation with proper indentation handling
- Dual interface: MCP tools for interactive use + programmatic API for scripting

## Development Commands

### Installation & Setup
```bash
# Install in development mode
pip install -e .

# Install from PyPI
pip install mcp-server-creator
```

### Running & Testing
```bash
# Run as MCP server
uvx mcp-server-creator
# OR
python mcp_server_creator/mcp_server_creator.py

# Run tests
python test_mcp_creator.py
```

### Important Environment Variables
- `FASTMCP_LOG_LEVEL`: Automatically set to `INFO` in the script. FastMCP 2.8.1+ requires uppercase values (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`).

### Package Management
```bash
# Build package
python -m build

# Version is managed in pyproject.toml (currently 0.1.1)
```

## MCP Tools Available

### Server Management
- `create_server(name, description, version)` - Creates new server config with auto-generated server_id
- `list_servers()` - Lists all in-memory server configurations  
- `get_server_details(server_id)` - Get full configuration details

### Tool Management  
- `add_tool(server_id, tool_name, description, parameters, return_type, implementation, is_async)` - Adds tools with parameter validation and import management

### Resource Management
- `add_resource(server_id, uri, name, description, mime_type, is_template, implementation)` - Adds static/dynamic resources with template support

### Code Generation
- `generate_server_code(server_id)` - Generates complete runnable Python code
- `save_server(server_id, filename)` - Exports generated code to file
- `create_example_server()` - Creates complete weather service example

## Key Implementation Details

- Server IDs are auto-generated from names using `name.lower().replace(" ", "_")`
- Import statements are automatically managed and deduplicated
- Code generation uses proper Python indentation (4 spaces)
- Template resources support parameter extraction from URI patterns (e.g., `{city}`)
- Both sync and async tool implementations are supported
- Generated servers are complete and runnable FastMCP applications

---
> Source: [GongRzhe/MCP-Server-Creator](https://github.com/GongRzhe/MCP-Server-Creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
