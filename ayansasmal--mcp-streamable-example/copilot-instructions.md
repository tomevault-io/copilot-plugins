## mcp-streamable-example

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server implementation using Node.js and TypeScript that provides database query capabilities through a `dbQueryTool`. The server uses DuckDB as the database engine and StreamableHttpTransport for HTTP-based MCP communication.

## Architecture

- **MCP Server**: Built with `@modelcontextprotocol/sdk` using StreamableHttpTransport
- **Database**: DuckDB with `@duckdb/node-api` (modern Neo client)
- **Transport**: HTTP-based streaming transport with Express.js
- **Data**: Employee CSV data loaded into DuckDB for querying

## Development Commands

### Essential Commands
```bash
# Install dependencies
npm install

# Development (uses Node.js native TypeScript support)
npm run dev

# Development fallback (if native TS support fails)
npm run dev:fallback

# Build TypeScript to JavaScript
npm run build

# Run production build
npm run start

# Clean build directory
npm run clean
```

### Node.js TypeScript Support
This project uses Node.js native TypeScript support via `--experimental-strip-types` flag. If this fails, use the fallback command with `ts-node-dev`.

## Project Structure

```
├── src/
│   ├── server.ts          # Main MCP server with StreamableHttpTransport
│   ├── tools/
│   │   └── dbQueryTool.ts # Database query tool implementation
│   ├── database/
│   │   └── db.ts          # DuckDB connection and operations
│   └── types/
│       └── index.ts       # TypeScript interfaces
├── data/
│   └── employees.csv      # Employee sample data
├── package.json
├── tsconfig.json
└── CLAUDE.md
```

## Key Components

### DatabaseManager (`src/database/db.ts`)
- Uses `@duckdb/node-api` Neo client for modern TypeScript support
- Loads CSV data into in-memory DuckDB instance
- Provides streaming query capabilities
- Includes SQL validation for security (only SELECT statements allowed)

### DbQueryTool (`src/tools/dbQueryTool.ts`)
- MCP tool that exposes database query functionality
- Streams results in chunks to handle large datasets
- Input validation and security measures
- Returns structured JSON responses

### MCP Server (`src/server.ts`)
- StreamableHttpTransport implementation with Express.js
- Handles MCP protocol requests (ListTools, CallTool)
- Provides health check and schema endpoints
- Graceful shutdown handling

## Database Schema

The `employees` table contains:
- `employeeId` (INTEGER): Unique employee identifier
- `employeeName` (VARCHAR): Full name of employee
- `location` (VARCHAR): Work location/city
- `startDate` (DATE): Employment start date

## Sample Queries

```sql
-- Basic queries
SELECT * FROM employees LIMIT 10;
SELECT COUNT(*) FROM employees;

-- Grouping and aggregation
SELECT location, COUNT(*) as employee_count
FROM employees
GROUP BY location
ORDER BY employee_count DESC;

-- Date filtering
SELECT * FROM employees
WHERE startDate >= '2020-01-01'
ORDER BY startDate DESC;
```

## Security Considerations

- Only SELECT statements are allowed in SQL queries
- Dangerous keywords (DROP, DELETE, UPDATE, INSERT, etc.) are blocked
- Results are limited to prevent excessive memory usage
- Input validation on all tool parameters

## Server Endpoints

When running, the server provides:
- **MCP Endpoint**: `http://localhost:3000/mcp` - Main MCP protocol endpoint
- **Health Check**: `http://localhost:3000/health` - Server status
- **Schema Info**: `http://localhost:3000/schema` - Database schema and sample queries

## Dependencies

### Production
- `@modelcontextprotocol/sdk`: MCP protocol implementation
- `@duckdb/node-api`: Modern DuckDB client with native TypeScript support
- `express`: HTTP server framework

### Development
- `typescript`: TypeScript compiler
- `ts-node-dev`: TypeScript execution for development (fallback)
- `@types/node` & `@types/express`: Type definitions

## Environment Variables

- `PORT`: Server port (default: 3000)
- `HOST`: Server host (default: localhost)

## Testing the Implementation

1. Start the server: `npm run dev`
2. Test health endpoint: `curl http://localhost:3000/health`
3. Get schema info: `curl http://localhost:3000/schema`
4. Use MCP client to connect to `http://localhost:3000/mcp`

## Common Development Patterns

- All database operations are async and use proper error handling
- Streaming is implemented for large query results
- TypeScript strict mode is enabled for type safety
- JSDoc comments are used for comprehensive documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ayansasmal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
