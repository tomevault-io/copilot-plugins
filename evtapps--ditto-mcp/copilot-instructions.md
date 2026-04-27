## ditto-mcp

> Best practices for tools, resources, schemas, and responses in the MCP server

# MCP Server Practices

## Tools

- Register tools in `createServer()` only; keep handlers small and deterministic.
- `inputSchema`: define with Zod field shapes (e.g., `{ param: z.string() }`) or a `z.object({ ... })` shape assigned to a constant and passed as fields. Keep numeric constraints (`int`, `positive`, upper bounds) and `url()` where applicable.
- Validate user input strictly (minLength, numeric constraints, URL where applicable).
- Return `{ content: [...] }` on success; `{ isError: true, content: [...] }` on failure. Keep error content concise and actionable.
- Error content should be concise, actionable, and safe (no secrets).

## Resources

- Use readable titles and descriptions. Keep resource bodies small and redacted if they include configuration. Only expose non-sensitive fields in resources like `ditto://config`.

## Guardrails

- Always enforce single-statement DQL and detect the first token (`SELECT|INSERT|UPDATE|DELETE|EVICT`). Strip comments before tokenizing.
- Enforce operation allow-list from config. Default to `SELECT` only.
- If configured, require the DQL to match at least one allow-list regex.

## Configuration Access

- Tool handlers do not read env directly. Resolve base URL, API key, and timeouts from the `ServerConfig` built in `loadConfig()`.

## Errors

- Prefer returning an error content block over throwing; throwing should be reserved for truly unexpected conditions.
- The Ditto client should convert failures into a Ditto-like envelope to keep tool handlers simple.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evtapps) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
