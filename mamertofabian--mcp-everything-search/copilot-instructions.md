## mcp-everything-search

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that provides cross-platform file search capabilities:
- **Windows**: Uses the Everything SDK for fast indexed search
- **macOS**: Uses `mdfind` (Spotlight) for native search
- **Linux**: Uses `locate`/`plocate` for filesystem search

The server exposes a single `search` tool with platform-specific parameter schemas and syntax support.

## Development Commands

### Testing and Running

```bash
# Run the server with MCP inspector for testing
npx @modelcontextprotocol/inspector uv run mcp-server-everything-search

# Run directly with uv
uv run mcp-server-everything-search

# Run as Python module (after pip install)
python -m mcp_server_everything_search
```

### Code Quality

```bash
# Type checking
uv run pyright

# Linting
uv run ruff check

# Formatting
uv run ruff format

# Run tests
uv run pytest
```

### Building

```bash
# Build the package
uv build
```

## Architecture

### Core Components

1. **server.py** - Main MCP server implementation
   - Defines the `search` tool with platform-specific schemas
   - Handles parameter parsing and validation using Pydantic
   - Routes search requests to appropriate platform provider
   - Returns formatted search results as TextContent

2. **search_interface.py** - Abstract search provider interface
   - `SearchProvider`: Abstract base class defining the search contract
   - `WindowsSearchProvider`: Everything SDK wrapper
   - `MacSearchProvider`: mdfind command wrapper
   - `LinuxSearchProvider`: locate/plocate command wrapper
   - `SearchResult`: Universal dataclass for all platforms

3. **platform_search.py** - Platform-specific parameter models
   - `BaseSearchQuery`: Common search parameters (query, max_results)
   - `WindowsSpecificParams`: Everything SDK options (match_path, match_case, etc.)
   - `MacSpecificParams`: mdfind options (live_updates, search_directory, etc.)
   - `LinuxSpecificParams`: locate options (ignore_case, regex_search, etc.)
   - `UnifiedSearchQuery`: Combines all parameter models with platform-aware schema generation
   - `build_search_command()`: Builds platform-specific command arrays

4. **everything_sdk.py** - Windows Everything SDK wrapper
   - `EverythingSDK`: ctypes wrapper for Everything64.dll
   - Comprehensive constant definitions for request flags and sort options
   - Windows filetime to Python datetime conversion
   - Full error handling with custom EverythingError exception

### Key Design Patterns

- **Strategy Pattern**: Platform-specific search providers implementing common SearchProvider interface
- **Factory Pattern**: `SearchProvider.get_provider()` returns the correct provider based on platform.system()
- **Adapter Pattern**: Each provider adapts platform-specific tools to unified SearchResult format

### Platform-Specific Notes

**Windows**:
- Requires `EVERYTHING_SDK_PATH` environment variable pointing to Everything64.dll
- Uses ctypes to call SDK functions directly (no subprocess calls)
- Supports full Everything search syntax with advanced filters and sorting

**macOS**:
- Uses subprocess to call `mdfind` command
- No additional dependencies required (built-in)

**Linux**:
- Checks for `plocate` first, falls back to `locate`
- Provides detailed error messages if database not initialized
- Uses subprocess to call locate commands

### Parameter Flow

1. Client sends search request with platform-specific parameters
2. `server.py` parses `base` and platform-specific params (e.g., `windows_params`)
3. Creates `UnifiedSearchQuery` from combined parameters
4. Routes to appropriate `SearchProvider` based on `platform.system()`
5. Provider executes search and returns list of `SearchResult` objects
6. Server formats results as TextContent with file details

## Environment Variables

- `EVERYTHING_SDK_PATH` (Windows only): Path to Everything64.dll (default: `D:\dev\tools\Everything-SDK\dll\Everything64.dll`)

## Installation Methods

The server supports three installation methods:
1. **Smithery CLI**: `npx -y @smithery/cli install mcp-server-everything-search --client claude`
2. **uv (recommended)**: Use `uvx mcp-server-everything-search`
3. **pip**: `pip install mcp-server-everything-search` then `python -m mcp_server_everything_search`

---
> Source: [mamertofabian/mcp-everything-search](https://github.com/mamertofabian/mcp-everything-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
