## screentime-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `screentime-mcp`, a Model Context Protocol (MCP) server that provides access to macOS Screen Time data. It loads Screen Time data from a SQLite database into DuckDB and exposes it via the `screentime_sql` tool for SQL queries.

## Build and Development Commands

All commands use [Task](https://taskfile.dev/). Common tasks:

- `task` or `task build` - Build the screentime-mcp binary
- `task clean` - Remove build artifacts
- `task tidy` - Run `go mod tidy`
- `task stdio-schema` - Extract MCP schema (for debugging)
- `task stdio-call-test` - Test a sample tool call

The binary is built to `bin/screentime-mcp`.

## Architecture

### Core Components

1. **Main Entry Point** (`main.go`): Handles CLI flags, logging setup, database initialization, and MCP server startup
2. **MCP Server** (`internal/mcp/`): Implements the Model Context Protocol server with STDIO and SSE transport options
3. **Database Layer** (`internal/db/`): Manages DuckDB connection and data transformation from Screen Time SQLite
4. **SQL Views** (`internal/db/duckdb_views.sql`): Comprehensive set of views for analyzing Screen Time data

### Key Architecture Patterns

- **Data Flow**: Screen Time SQLite → DuckDB → SQL Views → MCP Tool → LLM
- **Transport**: Supports both STDIO (default) and SSE transports for MCP
- **Database**: Uses DuckDB for analytical queries with pre-built views for common use cases
- **Timestamps**: Converts Apple Core Data timestamps (seconds since 2001-01-01) to standard Unix timestamps

### MCP Tool

The server exposes a single tool called `screentime_sql` that:
- Takes a `sql` parameter for DuckDB queries
- Returns results as CSV format
- Has access to extensive pre-built views (see `internal/mcp/screentime_sql.tooldesc.md`)

### Database Views Structure

Views are organized into categories:
- **Core Views**: Raw data access (v_app_usage, v_web_usage, v_notifications, etc.)
- **Summary Views**: Daily/weekly aggregations (v_daily_app_summary, v_top_apps_today, etc.)  
- **Analysis Views**: Patterns and insights (v_focus_sessions, v_app_transitions, etc.)
- **Web Views**: Web browsing categorization and analysis
- **Additional Views**: Bluetooth, app installs, location data, etc.

## Development Notes

### Database Access
- Requires "Full Disk Access" permissions on macOS to read Screen Time data
- Default database location: `~/Library/Application Support/Knowledge/knowledgeC.db`
- Uses read-only access mode in production
- Supports `:memory:` for testing/development

### Dev Mode
- Use `--dev` flag to continue running even if database connection fails
- Returns SQL errors upon tool call rather than exiting at startup
- Useful for development and testing without Screen Time data

### Configuration Options
- `--db`: Specify DuckDB file (default: `:memory:`)
- `--sse`: Use SSE transport instead of STDIO
- `--once`: Exit after one tool call (useful for testing)
- `--verbose`: Enable debug logging
- `--log-json`: JSON log format

## Dependencies

- Go 1.24.1+
- DuckDB (via go-duckdb bindings)
- MCP-Go library for protocol implementation
- pflag for CLI parsing

## Testing

Use `task stdio-call-test` to test the MCP tool with a sample SQL query. The schema can be inspected with `task stdio-schema | jq`.

---
> Source: [AgentDank/screentime-mcp](https://github.com/AgentDank/screentime-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
