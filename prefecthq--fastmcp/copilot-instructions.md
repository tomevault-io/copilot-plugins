## core-mcp-objects

> There are four major MCP object types:

There are four major MCP object types:

- Tools (src/tools/)
- Resources (src/resources/)
- Resource Templates (src/resources/)
- Prompts (src/prompts)

While these have slightly different semantics and implementations, in general changes that affect interactions with any one (like adding tags, importing, etc.) will need to be adopted, applied, and tested on all others. Note that while resources and resource templates are different objects, they are both in `src/resources/`.

---
> Source: [PrefectHQ/fastmcp](https://github.com/PrefectHQ/fastmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
