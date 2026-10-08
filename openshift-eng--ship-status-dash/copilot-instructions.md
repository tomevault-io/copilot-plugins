## ship-status-dev

> MCP server (ship-status-dev) for AI-callable dev tasks


Shared MCP server for AI-callable dev tasks (migrate, serve, test, monitor). Configuration and tools are in `ship-status-dev/server.py`.

When adding or modifying MCP tools, follow existing patterns in `ship-status-dev/server.py` (`_run_script_background`, `_run_foreground`, `_find_pids`, `_ensure_dev_log_dir`). Restart the MCP server after changes.

Dashboard API tools for agents live in the separate **`ship-status`** MCP (`mcp/`).

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
