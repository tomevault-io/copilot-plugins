## mcp-smart-proxy

> Token-efficient MCP server gateway with semantic tool search.

# MCP Smart Proxy

Token-efficient MCP server gateway with semantic tool search.

## When to use this skill

Use this skill when you need to:
- Proxy multiple MCP servers
- Dynamically manage server configurations
- Search tools semantically

## CLI

```bash
mcp-smart-proxy serve --config proxy.yaml
mcp-smart-proxy serve --config proxy.yaml --watch ./servers/
mcp-smart-proxy serve --config proxy.yaml --transport streamable-http
mcp-smart-proxy index --config proxy.yaml
mcp-smart-proxy status --config proxy.yaml
mcp-smart-proxy validate --config proxy.yaml
```

## Install

```bash
pip install -e .
```

---
> Source: [daedalus/mcp-smart-proxy](https://github.com/daedalus/mcp-smart-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
