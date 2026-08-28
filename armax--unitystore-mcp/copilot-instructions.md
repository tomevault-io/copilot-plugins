## unitystore-mcp

> See README.md for full documentation.

# Unity Asset Store MCP

See README.md for full documentation.

## Quick Dev Reference

- Runtime: Bun + TypeScript
- Entry: `src/index.ts` (MCP server with stdio transport)
- API client: `src/unity-api.ts`
- Extractor: `src/extractor.ts`
- Converter: `src/converter.ts`
- Never use `console.log` in this project — it corrupts the MCP stdio JSON-RPC protocol
- Auth flow: browser cookies → genesis token (`/api/auth/session`) → kharma session (`/login?user_access_token=`) → `X-Unity-Session` header

---
> Source: [Armax/unitystore-mcp](https://github.com/Armax/unitystore-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
