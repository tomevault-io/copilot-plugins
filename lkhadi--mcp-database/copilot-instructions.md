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

# Run in development mode (auto-reloads)
npm run dev -- --config config.json

# Run in production
npm start -- --config config.json

# Clean build artifacts
npm run clean
```

## Architecture

The server follows a modular architecture:

- **`src/index.ts`** - Main entry point. Initializes MCP server, loads config, creates connection manager, registers tools.
- **`src/config/`** - Configuration loading and Zod schema validation (`schema.ts`, `loader.ts`)
- **`src/db/`** - Database adapters and connection management
  - `connection-manager.ts` - Manages database connections with SSH tunnel support (lazy connection support)
  - `mysql-adapter.ts` / `postgresql-adapter.ts` - Database-specific implementations implementing `DatabaseAdapter` interface
- **`src/ssh/`** - SSH tunnel management (`tunnel-manager.ts`) for connecting through jump hosts
- **`src/tools/`** - MCP tool registration (`tool-generator.ts`) - registers 2 unified tools: `list_databases`, `execute_sql`
- **`src/utils/`** - Query validation and output formatting

## Key Design Patterns

- **Unified Tools**: Regardless of how many connections configured, only 2 tools are registered (`list_databases`, `execute_sql`). Connection ID is passed as a parameter.
- **Lazy SSH**: SSH tunnels can be configured with `lazy: true` to defer connection until first query.
- **Adapter Pattern**: `DatabaseAdapter` interface abstracts MySQL/PostgreSQL differences.
- **SSH Tunnel Manager**: Handles multihop jump hosts with automatic reconnection.

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
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
