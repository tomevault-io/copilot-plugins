## mcp-client-patterns

> MCP client usage patterns and troubleshooting


# MCP Client Patterns

## Lifecycle

The `McpPluginClient` uses a static factory pattern. Never construct directly:

```typescript
// Correct
const client = await McpPluginClient.connect({
  command: 'node',
  args: ['dist/index.js'],
  cwd: '/path/to/plugin',
  env: { MY_SERVICE_URL: 'http://localhost:3000' },
});

// Always disconnect when done
try {
  const tools = await client.listTools();
  const result = await client.callTool('my_tool', { query: 'test' });
} finally {
  await client.disconnect();
}
```

## One Client Per Suite

Create one MCP client per test suite, not per test. This avoids the 1-3 second startup overhead per test while maintaining isolation between suites.

## Error Handling

- `callTool()` does NOT throw on tool errors — check `result.isError`
- `callTool()` DOES throw on MCP protocol errors (timeout, connection lost)
- `disconnect()` is idempotent — safe to call multiple times
- If the child process crashes, all pending calls reject immediately

## Schema Conversion

When converting MCP tools to OpenAI format for LLM evals, use the allowlist to limit which tools the LLM sees:

```typescript
const toolDefs = convertToolsToArray(mcpTools, ['my_tool', 'my_other_tool']);
```

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
