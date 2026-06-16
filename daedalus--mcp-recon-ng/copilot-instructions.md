## mcp-recon-ng

> MCP server exposing full recon-ng OSINT framework functionality.

# MCP Recon-ng

MCP server exposing full recon-ng OSINT framework functionality.

## When to use this skill

Use this skill when you need to:
- Perform OSINT reconnaissance
- Search for domain/email information
- Use various recon modules
- Manage workspaces

## Tools

**Workspace:**
- `workspace_list`, `workspace_create`, `workspace_switch`, `workspace_delete`

**Module:**
- `module_list`, `module_load`, `module_unload`
- `module_options_list`, `module_options_set`, `module_run`

**Marketplace:**
- `marketplace_search`, `marketplace_install`, `marketplace_remove`

**Database:**
- `db_query`, `db_insert`, `db_delete`

**Utilities:**
- `run_command`, `get_info`

## Install

```bash
pip install mcp-recon-ng
pip install recon-ng
```

---
> Source: [daedalus/mcp-recon-ng](https://github.com/daedalus/mcp-recon-ng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
