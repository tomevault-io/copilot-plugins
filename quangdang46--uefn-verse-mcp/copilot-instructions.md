## uefn-verse-mcp

> This is uefn-mcp, an MCP server for UEFN. Key info:

# AGENTS.md

## For AI Agents

This is uefn-mcp, an MCP server for UEFN. Key info:

- **Architecture:** Two-process MCP (external server + in-UEFN listener)
- **Tools:** ~40 direct commands + `run_tool()` escape hatch for the full uefn_tools registry (see `uefn_tools.__tool_count__`)
- **Setup:** `python deploy.py` → `import uefn_tools as ut; ut.run("mcp_start")`
- **Rules:** All `unreal.*` calls must run on UEFN main thread (queue + tick callback)
- **MCP throttle:** Bridge drains **one** queued command per Slate tick by default (`UEFN_MCP_TICK_BATCH_LIMIT=1`), caps backlog (`UEFN_MCP_MAX_QUEUE_DEPTH`), HTTP **503** when full. External Python client serializes sends (`server/bridge.py` lock) + optional `UEFN_MCP_CLIENT_COOLDOWN_SEC`.

---
> Source: [quangdang46/uefn-verse-mcp](https://github.com/quangdang46/uefn-verse-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
