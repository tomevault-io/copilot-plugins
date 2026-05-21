## project-overview

> This project provides an MCP (Model Context Protocol) server implementation for database connections. It allows LLM-powered applications to connect to and query databases through the MCP protocol.

# MCP Database Server Project Overview

## Project Purpose
This project provides an MCP (Model Context Protocol) server implementation for database connections. It allows LLM-powered applications to connect to and query databases through the MCP protocol.

## Key Technologies
- TypeScript
- Node.js
- Model Context Protocol (MCP)
- Database Connections (SQLite, PostgreSQL, SQL Server)
- ESM Modules

## Architecture

The project follows a modular architecture:

### Core Components
- **Database Interfaces**: Defines the common interface that all database implementations must follow.
- **Database Implementations**: Concrete implementations for each supported database type.
- **MCP Server**: The main server that handles MCP protocol communication.
- **Factory**: Creates database instances based on configuration.
- **Tools**: MCP tool implementations for database operations.
- **Resources**: MCP resource implementations for database schema information.

### Directory Structure

- `src/`: Source code
  - `interfaces/`: TypeScript interfaces for the codebase
  - `databases/`: Database implementations
  - `tools/`: MCP tool implementations
  - `resources/`: MCP resource implementations
  - `server.ts`: Main MCP server implementation
  - `factory.ts`: Database factory
  - `index.ts`: Entry point and exports

- `test/`: Test files
- `dist/`: Compiled JavaScript output
- `assets/`: Static assets

## Important Development Notes

1. The project uses ES modules (ESM) with `.js` extensions in import paths.
2. Some dependencies (like 'pg') are CommonJS modules and require special import handling.
3. All database implementations follow the same interface for consistency.
4. The server supports both STDIO and SSE transports for different use cases.

## Common Tasks

- **Adding a new database type**: Create a new implementation in `src/databases/` following the Database interface, then update the factory.
- **Adding new MCP tools**: Add to the tools directory and register in the server.
- **Running in development**: Use `npm run dev` to start the TypeScript compiler in watch mode.
- **Testing**: Run `npm test` to execute the Jest test suite. 

---
> Source: [cuongtl1992/mcp-dbs](https://github.com/cuongtl1992/mcp-dbs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
