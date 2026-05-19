## http-transport

> - The MCP server can run in HTTP mode using the `--http` CLI flag.

# HTTP Transport Mode for MCP Server

- The MCP server can run in HTTP mode using the `--http` CLI flag.
- When `--http` is set, the entrypoint [src/index.ts](mdc:src/index.ts) starts an Express server and exposes the MCP API at `/mcp`.
- Each POST to `/mcp` creates a new `McpServer` and `StreamableHTTPServerTransport` for stateless, isolated handling.
- The HTTP port can be set with `--port` (default: 3000).
- Express must be installed as a dependency for HTTP mode.
- If `--http` is not set, the server runs in stdio mode as before.

---
> Source: [nloui/paperless-mcp](https://github.com/nloui/paperless-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
