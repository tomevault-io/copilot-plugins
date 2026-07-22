## open-ux-tools

> This document describes the purpose of the key configuration files in this package that relate to MCP server distribution and plugin registries.

# AI Agent Guidelines for `@sap-ux/fiori-mcp-server`

This document describes the purpose of the key configuration files in this package that relate to MCP server distribution and plugin registries.

## Configuration Files

### `server.json`

This file is the **MCP server registry manifest**. It conforms to the [MCP server schema](https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json) and is used to list this server in MCP server registries (e.g., the official MCP server registry at `registry.modelcontextprotocol.io`).

Key fields:
- `name` — Unique server identifier in reverse-domain notation (`io.github.SAP/fiori-mcp-server`)
- `description` — Short description shown in registry listings
- `repository` — Points to the GitHub source repository and subfolder
- `version` — Must be kept in sync with `package.json`
- `packages[].environmentVariables` — Documents environment variables users can configure (e.g., `LOG_LEVEL`, `SAP_UX_FIORI_TOOLS_DISABLE_TELEMETRY`)

> **Important:** Do not manually update `version` in `server.json` or `.claude-plugin/plugin.json`. The `scripts/sync-mcp-manifests.cjs` script runs automatically in the CI/CD pipeline (`version` job in `pipeline.yml`) after changesets bump `package.json`, and keeps all three files in sync.

### `.claude-plugin/plugin.json`

This file is the **Claude Code plugin manifest**. It registers this MCP server as a plugin in the [Claude Code plugin registry](https://code.claude.com/docs/en/plugins), allowing users to discover and install it from within Claude Code.

Key fields:
- `name` — Plugin identifier shown in the Claude Code registry
- `description` — Brief explanation of what the plugin does (shown to users in registry)
- `author` — Organization name and URL
- `homepage` — URL to the package folder (documentation and README)
- `repository` — URL to the source code repository root
- `license` — SPDX license identifier
- `keywords` — Discovery tags used for search in the registry

The `.mcp.json` file at the plugin root is automatically picked up by Claude Code to configure the bundled MCP server when the plugin is enabled.

### `.mcp.json`

This file is the **project-scoped MCP server configuration** for Claude Code. It follows the [Claude Code `.mcp.json` format](https://code.claude.com/docs/en/mcp#option-1-exclusive-control-with-managed-mcp-json) and serves two purposes:

1. **Plugin bundling** — When this package is installed as a Claude Code plugin (via `.claude-plugin/plugin.json`), Claude Code reads `.mcp.json` to automatically start the MCP server.
2. **Local development** — Developers working in this repository can use this file directly to connect the locally published server to Claude Code.

Key fields:
- `mcpServers.fiori-mcp.type` — Transport type (`stdio` for local process)
- `mcpServers.fiori-mcp.command` / `args` — How to launch the server via `npx`

---
> Source: [SAP/open-ux-tools](https://github.com/SAP/open-ux-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
