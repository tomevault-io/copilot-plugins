## mysql-mcp-server

> This file provides setup and context for GitHub Copilot.

# Copilot Instructions

This file provides setup and context for GitHub Copilot.

## Setup Steps

These steps should be run to initialize the environment before the sandbox firewall is applied.

\`\`\`setup-steps
go mod download
go build ./...
\`\`\`

## Architecture Context

- This is an MCP (Model Context Protocol) server for MySQL.
- It supports read-only operations by default.
- It uses a custom ConnectionManager to handle multiple DSNs and SSH tunnels.
- Most tools follow a \`wrapTool\` pattern for consistent logging and token tracking.

---
> Source: [askdba/mysql-mcp-server](https://github.com/askdba/mysql-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
