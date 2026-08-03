## dynatrace-for-ai

> This plugin includes skills as well as the Dynatrace MCP server for Dynatrace platform access.

# Dynatrace Plugin

This plugin includes skills as well as the Dynatrace MCP server for Dynatrace platform access.

## Required setup for Dynatrace MCP Server

Set these environment variables, e.g., in `~/.claude/settings.json` under `env`:

- `DT_ENVIRONMENT` — your Dynatrace environment URL, e.g. `https://abc12345.apps.dynatrace.com` (Note: `https://` is required)
- `DT_PLATFORM_TOKEN` — a platform token with at least MCP gateway scopes

Consult the [Dynatrace MCP server docs
](https://docs.dynatrace.com/docs/shortlink/dynatrace-mcp-server) for a full list of scopes required for using the MCP Server with a Platform Token.

## Skills for Observability related questions

Please use the included skills for observability related questions.

---
> Source: [Dynatrace/dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
