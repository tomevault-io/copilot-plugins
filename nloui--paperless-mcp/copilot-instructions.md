## typescript-mcp-migration

> - The main entrypoint is [src/index.ts](mdc:src/index.ts), which starts an MCP server for Paperless-NGX using the @modelcontextprotocol/sdk package.

# TypeScript MCP Server Migration

- The main entrypoint is [src/index.ts](mdc:src/index.ts), which starts an MCP server for Paperless-NGX using the @modelcontextprotocol/sdk package.
- All core logic is now in TypeScript; all main files in src/ use the .ts extension.
- The MCP server is started using `McpServer` and `StdioServerTransport` from the SDK, with `await server.connect(transport)`.
- The project can be run directly in TypeScript using the npm script: `npm run start -- <baseUrl> <token>` (see [package.json](mdc:package.json)).
- Node.js types are provided by @types/node in devDependencies.

---
> Source: [nloui/paperless-mcp](https://github.com/nloui/paperless-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
