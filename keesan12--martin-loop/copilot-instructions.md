## martin-loop

> - Martin Loop ships both a repo-wide OSS/RC surface and a standalone publishable MCP package at `packages/mcp`.

# AGENTS.md

## Project Overview
- Martin Loop ships both a repo-wide OSS/RC surface and a standalone publishable MCP package at `packages/mcp`.

## Build and Verify
- For MCP-only changes, run `pnpm --filter @martinloop/mcp test`, `pnpm --filter @martinloop/mcp build`, and `pnpm --filter @martinloop/mcp smoke:pack`.
- For release-surface or packaging changes that could affect CI, run `pnpm release:matrix:local`.

## MCP Registry Guardrails
- Do not call `packages/mcp` registry-ready unless `packages/mcp/package.json` includes `mcpName` and `packages/mcp/server.json` exists with matching `name`, `version`, and npm package `identifier`.
- npm publication happens before official MCP Registry publication.
- The official MCP Registry flow runs from `packages/mcp`: `mcp-publisher login github`, then `mcp-publisher publish`.
- The current official registry server name for the public MCP package is `io.github.keesan12/martin-loop`.

---
> Source: [Keesan12/martin-loop](https://github.com/Keesan12/martin-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
