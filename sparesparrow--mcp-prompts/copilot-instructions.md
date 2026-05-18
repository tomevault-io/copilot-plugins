## server

> - Use @modelcontextprotocol/sdk latest

# MCP Server Application Rules

## MCP SDK Integration
- Use @modelcontextprotocol/sdk latest
- Register tools with server.tool()
- Register resources with server.resource()
- Use Zod for schema validation

## Composition Root
- Wire all dependencies in index.ts
- Use dependency injection
- Configure all adapters
- Handle startup/shutdown gracefully

## Transport Configuration
- Support stdio and HTTP transports
- Configure SSE for remote access
- Handle connection lifecycle
- Implement health checks

## Error Handling
- Structured error responses
- Proper logging with Pino
- Graceful degradation
- Circuit breaker patterns # MCP Server Application Rules

---
> Source: [sparesparrow/mcp-prompts](https://github.com/sparesparrow/mcp-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
