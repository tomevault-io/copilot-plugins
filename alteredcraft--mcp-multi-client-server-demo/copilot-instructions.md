## mcp-multi-client-server-demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) Chat application - a CLI tool for interactive conversations with Claude through the Anthropic API. It's based on Anthropic's MCP course (https://anthropic.skilljar.com/introduction-to-model-context-protocol).

## Project Structure

```
├── main.py                    # Entry point - initializes MCP clients and CLI
├── mcp_clients/              # Directory containing MCP client implementations
│   ├── mcp_client_console.py   # Console/stdio MCP client
│   └── README.md             # Guide for adding new client types
├── mcp_servers/              # Directory containing MCP servers
│   ├── documents_mcp_server.py  # Document management server
│   ├── calculator_mcp_server.py # Mathematical operations server
│   └── README.md             # Guide for adding new servers
├── core/                     # Core application modules
│   ├── claude.py            # Claude API wrapper
│   ├── cli.py               # CLI application and UI
│   ├── cli_chat.py          # Chat interface logic
│   ├── chat.py              # Core chat functionality
│   └── tools.py             # Tool implementations
└── docs/                     # Educational materials and slides
```

## Key Commands

### Running the Application
```bash
# Run the main application
uv run main.py

# To run with additional MCP servers
uv run main.py mcp_servers/calculator_mcp_server.py mcp_servers/weather_mcp_server.py
```

### Installation
```bash
# This project requires uv
uv sync
```

## Multi-Server Architecture

The application supports flexible MCP server loading with two modes:

### Auto-Discovery Mode (Default)
When no servers are specified, automatically loads all `*mcp_server.py` files in the `mcp_servers/` directory:

```bash
# Auto-discover and load all available servers
uv run main.py
```

### Manual Selection Mode  
Specify exactly which servers to load:

```bash
# Load only calculator server
uv run main.py mcp_servers/calculator_mcp_server.py

# Load multiple specific servers
uv run main.py mcp_servers/calculator_mcp_server.py mcp_servers/weather_mcp_server.py
```

### Server Status Display
The application displays which servers are loaded:
- 🔍 Auto-discovery mode shows the search pattern
- 📋 Manual mode shows "Using specified MCP servers"  
- 🚀 Lists all loaded servers with friendly names

This architecture allows for:
1. **Easy development** - just add new `*mcp_server.py` files and they're auto-discovered
2. **Flexible deployment** - choose exactly which servers to run
3. **Clear visibility** - see exactly what's loaded at startup

### Key Design Patterns

- **MCP Protocol**: Uses Model Context Protocol for extensible tool integration
- **Document Retrieval**: `@` mentions trigger document retrieval from MCP server
- **Command System**: `/` prefix executes MCP-defined prompts/commands
- **Async Architecture**: Built on asyncio for concurrent MCP server connections

### Environment Configuration

Required `.env` file:
```
ANTHROPIC_API_KEY=""  # Required: Your Anthropic API key
CLAUDE_MODEL=""       # Required: Model ID (e.g., claude-3-sonnet-20240229)

# Optional: Proxy configuration for corporate environments
HTTP_PROXY=http://127.0.0.1:9000
HTTPS_PROXY=http://127.0.0.1:9000
NO_PROXY=localhost,127.0.0.1,.local
VERIFY_SSL=false  # Set to false for corporate proxies with SSL issues
```

### Adding Features

1. **New Documents**: Add to `docs` dictionary in `mcp_servers/documents_mcp_server.py`
2. **New MCP Tools**: Implement in MCP server files following MCP protocol
3. **New Commands**: Add prompts to MCP server's prompt list
4. **New MCP Servers**: Create new server files in the `mcp_servers/` directory

### Development Notes

- No linting or type checking currently configured
- Windows requires `asyncio.WindowsProactorEventLoopPolicy()` (handled in main.py:63-64)
- Project uses modern Python packaging with `pyproject.toml`
- Supports Python 3.10+ (per pyproject.toml)

---
> Source: [AlteredCraft/mcp_multi_client_server_demo](https://github.com/AlteredCraft/mcp_multi_client_server_demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
