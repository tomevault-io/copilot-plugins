## mcp-memory

> This is a memory server for MCP (Multi-Client Protocol) that uses PostgreSQL with pgvector for vector similarity search.

# CLAUDE.md Guidelines

## Project Structure
This is a memory server for MCP (Multi-Client Protocol) that uses PostgreSQL with pgvector for vector similarity search.

## Commands
- `npm start`: Start server with Node.js
- `npm run dev`: Start with nodemon for development (auto-reload)
- `npm run prisma:migrate`: Initialize database (mentioned in README but not in package.json)

## Code Style
- Use ES Module imports (type: "module")
- camelCase for variables and functions
- Descriptive naming with clear purpose
- Structured error handling with try/catch and detailed logging
- JSON-RPC response format with proper error codes

## Database
- PostgreSQL with pgvector extension
- Use UUID primary keys with gen_random_uuid()
- Store JSON content with JSONB type
- Include created_at/updated_at timestamps

## Best Practices
- Use sendLogMessage() for consistent logging with levels
- Document functions with clear comments
- Include parameter descriptions in method definitions
- Validate inputs before use
- Use environment variables via dotenv for configuration

---
> Source: [sdimitrov/mcp-memory](https://github.com/sdimitrov/mcp-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
