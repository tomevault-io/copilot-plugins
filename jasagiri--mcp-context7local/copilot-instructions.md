## mcp-context7local

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview
Context7 Local is an on-premises MCP (Model Context Protocol) server for managing private/internal library documentation. It's written in Nim and provides a self-hosted alternative to cloud-based documentation services.

## Essential Commands

### Development
```bash
# Build the project
nimble build

# Run in development mode
nimble dev

# Run tests
nimble test                    # All tests
nimble test_comprehensive      # Full test suite with 100% coverage
nimble test_coverage_report    # Generate coverage report

# Run specific test file
nim c -r tests/test_library_manager.nim

# Clean build artifacts
nimble clean
```

### Running the Server
```bash
# Start MCP server (stdio mode)
./context7local server

# Initialize configuration
./context7local init

# Register a library
./context7local register <name> <version> <docs_file>
```

## Architecture

The codebase follows an async/await pattern with modular separation:

- **`src/context7local.nim`**: Main server implementing MCP protocol. Handles tool calls and resource requests.
- **`src/library_manager.nim`**: Core business logic for library CRUD operations, search indexing, and JSON persistence.
- **`src/config_manager.nim`**: YAML configuration management with validation and defaults.
- **`src/cli.nim`**: Command-line interface for administrative tasks.
- **`src/mcp_helpers.nim`**: Utility functions for building MCP protocol responses.

### Key Design Patterns
1. **Async Operations**: All I/O operations use async/await for non-blocking execution
2. **File-based Storage**: Libraries stored as JSON files in configurable data directory
3. **In-memory Search Index**: Keyword-based search rebuilt on startup from persisted data
4. **Dual Operation Modes**: Functions as both CLI tool and MCP server

## MCP Protocol Implementation

The server exposes three main tools:
- `register_library`: Add new library documentation
- `search_libraries`: Search by name/description
- `get_library_docs`: Retrieve specific library docs

Transport modes supported: stdio, http, sse

## Testing Strategy

Tests use a comprehensive approach with temporary directories for isolation:
- Unit tests for each module
- Integration tests for MCP protocol
- Error handling tests for edge cases
- 100% code coverage target

Run tests before committing changes to ensure nothing breaks.

---
> Source: [jasagiri/mcp-context7local](https://github.com/jasagiri/mcp-context7local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
