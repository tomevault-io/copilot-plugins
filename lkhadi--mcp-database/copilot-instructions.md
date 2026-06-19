## mcp-database

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Database Access Server - A Model Context Protocol (MCP) server that provides secure access to MySQL and PostgreSQL databases with unified tools.

## Commands

```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Run in development mode
npm run dev -- --config config.json

# Run in production
npm start -- --config config.json

# Run tests
npm test

# Clean build artifacts
npm run clean
```

## Architecture

The server follows a modular architecture:

- **`src/index.ts`** - Main entry point. Initializes MCP server, loads config, creates connection manager, registers tools.
- **`src/config/`** - Configuration loading and Zod schema validation (`schema.ts`, `loader.ts`)
- **`src/db/`** - Database adapters, connection management, and retry logic
  - `connection-manager.ts` - Manages database connections with SSH tunnel support, lazy connections, and SSH-backed recovery
  - `mysql-adapter.ts` / `postgresql-adapter.ts` - Database-specific implementations of the `DatabaseAdapter` interface, with transient-error retry
  - `connection-retry.ts` - Classifies transient connection errors and provides the `withConnectionRetry` helper
  - `types.ts` - Shared adapter interfaces and types
- **`src/ssh/`** - SSH tunnel management (`tunnel-manager.ts`) for connecting through jump hosts
- **`src/tools/`** - MCP tool registration (`tool-generator.ts`) - registers 2 unified tools: `list_databases`, `execute_sql`
- **`src/utils/`** - Query validation and output formatting

## Key Design Patterns

- **Unified Tools**: Regardless of how many connections configured, only 2 tools are registered (`list_databases`, `execute_sql`). Connection ID is passed as a parameter.
- **Lazy SSH**: SSH tunnels can be configured with `lazy: true` to defer connection until first query.
- **Adapter Pattern**: `DatabaseAdapter` interface abstracts MySQL/PostgreSQL differences.
- **SSH Tunnel Manager**: Handles multihop jump hosts with automatic reconnection.
- **Connection Resilience**: Adapters retry once on transient connection errors; SSH-backed connections trigger tunnel reconnection and adapter recreation on failure.
- **Tests**: Unit/integration tests live in `tests/` and run with `node --test` via `npm test`.

## Configuration

See `config.example.json` for reference. Key fields:
- `connections[].id` - Must be lowercase alphanumeric with underscores
- `connections[].databases` - Use `"*"` for auto-discovery, or array for specific databases
- `ssh` - Optional, supports `jumpHosts` array for multihop

## Available Tools

| Tool | Description |
|------|-------------|
| `list_databases` | List all configured connections and their databases |
| `execute_sql` | Execute SQL on specified connection/database |

---
> Source: [lkhadi/mcp-database](https://github.com/lkhadi/mcp-database) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
