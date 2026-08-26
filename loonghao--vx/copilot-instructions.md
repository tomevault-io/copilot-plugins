## vx-mcp

> Rules for configuring MCP servers with vx — replace npx/uvx with vx for zero-config tool management


# VX MCP Integration Rules

When configuring MCP (Model Context Protocol) servers, **always** use `vx` instead of `npx` or `uvx`:

## Pattern

```json
{
  "mcpServers": {
    "server-name": {
      "command": "vx",
      "args": ["npx", "-y", "@scope/package@latest"]
    }
  }
}
```

## Benefits

- Users don't need Node.js/Python pre-installed
- Same config works across Windows, macOS, Linux
- Tool versions managed by vx.toml if present

## Migration

| Replace | With |
|---------|------|
| `"command": "npx"` | `"command": "vx", "args": ["npx", ...]` |
| `"command": "uvx"` | `"command": "vx", "args": ["uvx", ...]` |
| `"command": "node"` | `"command": "vx", "args": ["node", ...]` |
| `"command": "python"` | `"command": "vx", "args": ["python", ...]` |

## Common MCP Servers with vx

```json
{
  "filesystem": { "command": "vx", "args": ["npx", "-y", "@modelcontextprotocol/server-filesystem", "/path"] },
  "github": { "command": "vx", "args": ["npx", "-y", "@modelcontextprotocol/server-github"] },
  "sqlite": { "command": "vx", "args": ["uvx", "mcp-server-sqlite", "--db-path", "db.sqlite"] }
}
```

---
> Source: [loonghao/vx](https://github.com/loonghao/vx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
