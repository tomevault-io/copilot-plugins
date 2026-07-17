## oracle-mcp-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Oracle MCP Server that provides a flexible interface for AI assistants to interact with Oracle databases through the Model Context Protocol. The server supports multiple schemas and provides comprehensive database introspection and query capabilities.

## Common Development Commands

### Running the Server
```bash
# Install dependencies
npm install

# Start the MCP server
npm start

# Run in development mode with file watching
npm run dev
```

### Testing Connection
```bash
# Set environment variables first
export ORACLE_CONNECTION_STRING=hostname:1521/service_name
export ORACLE_USER=your_username
export ORACLE_PASSWORD=your_password

# Run the server
npm start
```

## Architecture

### MCP Server Implementation
- **Framework**: Model Context Protocol (MCP) SDK
- **Database**: Oracle Database with `oracledb` client library
- **Connection**: Supports multiple authentication methods (Easy Connect, TNS, individual components)
- **Transport**: stdio (stdin/stdout for JSON-RPC communication)

### Core Components

1. **Main Server** (`src/index.js`):
   - Extends MCP Server with Oracle database integration
   - Implements 6 tools for database operations
   - Supports multiple Oracle connection methods
   - Automatically converts PostgreSQL-style parameters to Oracle bind variables

2. **Available Tools**:
   - `execute_query` - Execute any SQL query with parameter binding (supports SELECT, INSERT, UPDATE, DELETE, etc.)
   - `list_tables` - List tables from specified schema or all accessible schemas
   - `describe_table` - Get table structure and column details (searches all schemas if not specified)
   - `get_table_indexes` - View indexes on a specific table (searches all schemas if not specified)
   - `get_table_constraints` - View constraints (PK, FK, unique, check) for a table
   - `list_schemas` - List all accessible schemas in the database

### Connection Methods

The server supports three Oracle connection methods:

1. **Easy Connect String**:
   ```env
   ORACLE_CONNECTION_STRING=hostname:1521/service_name
   ORACLE_USER=username
   ORACLE_PASSWORD=password
   ```

2. **TNS Name** (requires tnsnames.ora):
   ```env
   ORACLE_TNS_NAME=ORCL
   ORACLE_USER=username
   ORACLE_PASSWORD=password
   ```

3. **Individual Components**:
   ```env
   ORACLE_HOST=localhost
   ORACLE_PORT=1521
   ORACLE_SERVICE_NAME=ORCL  # or ORACLE_SID=ORCL
   ORACLE_USER=username
   ORACLE_PASSWORD=password
   ```

### Security Features

1. **Connection Management**: Single connections per query (no persistent pools by default)
2. **Parameter Binding**: All queries use bind variables to prevent SQL injection
3. **Schema Flexibility**: Can query across multiple schemas or restrict to specific schema
4. **Auto-commit**: Enabled by default for DML operations
5. **Audit Logging**: All queries are logged with timestamp and duration for security auditing

### Claude Desktop Configuration

To use this server with Claude Desktop, add to your config:

```json
{
  "mcpServers": {
    "oracle": {
      "command": "node",
      "args": ["src/index.js"],
      "cwd": "/path/to/oracle-mcp",
      "env": {
        "ORACLE_CONNECTION_STRING": "${ORACLE_CONNECTION_STRING}",
        "ORACLE_USER": "${ORACLE_USER}",
        "ORACLE_PASSWORD": "${ORACLE_PASSWORD}"
      }
    }
  }
}
```

### Environment Variables

Required environment variables (set in `.env`):
- Connection method (one of):
  - `ORACLE_CONNECTION_STRING` + credentials
  - `ORACLE_TNS_NAME` + credentials  
  - `ORACLE_HOST`, `ORACLE_PORT`, `ORACLE_SERVICE_NAME`/`ORACLE_SID` + credentials
- `ORACLE_USER`: Database username
- `ORACLE_PASSWORD`: Database password

Optional:
- `ORACLE_DEFAULT_SCHEMA`: Default schema if different from user
- `ORACLE_CLIENT_PATH`: Path to Oracle Instant Client (if using thick mode)

## Development Notes

- Node.js 18+ required (uses ES modules)
- The `oracledb` driver runs in "thin mode" by default (no Oracle Client needed)
- For advanced features, install Oracle Instant Client and uncomment `oracledb.initOracleClient()`
- Parameter conversion: PostgreSQL-style `$1, $2` automatically converted to Oracle `:1, :2`
- All table/schema names are automatically uppercased (Oracle convention)

## Oracle-Specific Considerations

- **Case Sensitivity**: Oracle object names are uppercase by default
- **Bind Variables**: Uses numbered bind variables (`:1, :2`) instead of `$1, $2`
- **LISTAGG**: Used for concatenating multiple rows (Oracle equivalent of `string_agg`)
- **Data Dictionary Views**: Uses `ALL_*` views for metadata queries
# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

---
> Source: [git-scarrow/oracle-mcp-server](https://github.com/git-scarrow/oracle-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
