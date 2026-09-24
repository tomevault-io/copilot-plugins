## skylight-mcp

> Read `CLAUDE.md` for build, test, authentication, and release conventions.

# Development notes

Read `CLAUDE.md` for build, test, authentication, and release conventions.

## OAuth redirects

`login()` must resolve relative authorization redirects against the current request URL. Only the authentication origin may receive the session cookie. Read the authorization code only from the exact configured callback origin and path, without fetching that callback. Reject URL credentials, fragments, HTTPS downgrades, and other destinations. Do not include redirect URLs in errors because their query strings can contain credentials.

Tests cover these cases in `tests/auth-session-login.test.ts`. Run `npm run test:coverage` and `npm run build` after authentication changes. A read of `/frames` verified password login on September 18, 2026. Repeat that read before claiming compatibility with a later API change.

## MCP v2

Use `@modelcontextprotocol/server` for the server and `@modelcontextprotocol/client` for clients. `@chrischall/mcp-utils` 0.28 supplies the server factory and test helpers. Use `serveStdio` from the official SDK with `createMcpServer`; the shared `runMcp` helper still connects the legacy transport and does not handle modern stdio discovery. Tool schemas must be complete Zod objects, rather than raw shapes. Schema inspection tests read `.shape` directly.

`tests/protocol.test.ts` exercises modern discovery, tool listing, schema validation, credential-free startup, and legacy initialization through the official HTTP adapter. The production entry point remains stdio. A modern request must include protocol version, client capabilities, and client identity metadata. Match the method and tool-name headers to the request.

Run `npm run test:coverage` and `npm run build`. Keep all version markers unchanged unless the release workflow updates them.

The test commands build first so `tests/stdio.test.ts` exercises the production entry point.

---
> Source: [chrischall/skylight-mcp](https://github.com/chrischall/skylight-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
