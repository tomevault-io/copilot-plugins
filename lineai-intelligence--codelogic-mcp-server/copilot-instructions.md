## about-codebase

> General information about the CodeLogic MCP Server codebase and its purpose

- This repository contains a Model Context Protocol (MCP) server that integrates with CodeLogic's knowledge graph APIs
- It enables AI programming assistants to access dependency data from CodeLogic to analyze code and database impacts
- Tools: `codelogic-method-impact`, `codelogic-database-impact`, and `codelogic-graph-*` (capabilities, search, impact, path-explain, validate-change-scope, owners)
- The core package is in `src/codelogic_mcp_server/` with `server.py`, `handlers/` (package), and `utils.py`

---
> Source: [lineai-intelligence/codelogic-mcp-server](https://github.com/lineai-intelligence/codelogic-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
