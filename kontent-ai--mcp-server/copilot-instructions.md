## mcp-tool-naming

> Enforce naming conventions for Kontent.ai MCP tools

When creating or modifying MCP tools, follow these naming conventions:

- Format: `[action]-[entity]`
- Use full entity names: `content-type`, `content-type-snippet`, `content-item`, `content-item-variant`, `taxonomy-group`
- For list tools filtered by another entity: `list-[entity]-by-[filter-entity]` (e.g., `list-content-item-variants-by-collection`)
- Examples: `get-content-type`, `list-content-item-variants`, `patch-content-type-snippet`

---
> Source: [kontent-ai/mcp-server](https://github.com/kontent-ai/mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
