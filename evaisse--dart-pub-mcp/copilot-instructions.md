## dart-pub-mcp

> This document explains how to connect MCP-compatible agents (Claude Desktop, Claude Code, Cursor, MCP Inspector, etc.) to **dart-pub-mcp** and effectively use the exposed resources.

# Agent Integration Guide

This document explains how to connect MCP-compatible agents (Claude Desktop, Claude Code, Cursor, MCP Inspector, etc.) to **dart-pub-mcp** and effectively use the exposed resources.

## Transport

The server speaks the Model Context Protocol over **stdio**. Launch the server and point your agent at the executable (Node.js script) via its MCP configuration.

```bash
npm install
npm start
```

Most clients allow registering a custom MCP server by specifying:

- **Name:** `dart-pub-mcp`
- **Transport:** `stdio`
- **Command:** `npm`
- **Arguments:** `["start"]`
- **Environment variables (optional):**
  - `PUB_CACHE` – set if your pub cache lives somewhere other than `~/.pub-cache`.

## Resource catalogue

Once connected, the agent can list and read the following resource templates:

| Template | Description |
| --- | --- |
| `pub.dev://package/{package}` | Package summary from pub.dev (mirrors `/api/packages/{package}`) |
| `pub.dev://package/{package}/version/{version}` | Version metadata (`/api/packages/{package}/versions/{version}`), `version=latest` allowed |
| `pub.dev://package/{package}/score` | Score metrics (`/api/packages/{package}/score`) |
| `pub.dev://package/{package}/publisher` | Publisher info (`/api/packages/{package}/publisher`) |
| `pub.dev://package/{package}/version/{version}/files` | JSON list of files in the published archive |
| `pub.dev://package/{package}/version/{version}/file/{+filePath}` | Raw file contents from the published archive |
| `pubcache://package/{package}/versions` | Versions discovered in the local pub cache |
| `pubcache://package/{package}/version/{version}/files` | Files available locally for a specific version |
| `pubcache://package/{package}/version/{version}/file/{+filePath}` | Raw file read out of the local pub cache |

The `pub.dev://package/{package}` template auto-completes package names using the official `/api/package-name-completion-data` endpoint.

## Usage tips

- To surface release notes quickly, read `pub.dev://package/<name>/version/<version>/file/CHANGELOG.md`.
- Prefer local cache URIs (`pubcache://…`) when you need to inspect packages already downloaded on disk; remote reads may require network access.
- For headless clients, list resources first to discover URI patterns: `listResources` with template variables will show candidate package names.
- Combine `pub.dev` metadata (scores, publisher details) with local source inspection to build richer prompts for the LLM.

## Troubleshooting

- **Missing files:** The remote archive resource only includes files published to pub.dev. If a path is absent, verify the version actually ships it.
- **Local cache not found:** Ensure the `PUB_CACHE` environment variable points to the correct directory or the default `~/.pub-cache` exists.
- **Network restrictions:** The server downloads archives on demand. If running in an offline environment, rely on the local cache resources instead.

For more advanced integration scenarios, consult the [official MCP specification](https://modelcontextprotocol.io/specification/latest).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evaisse)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/evaisse)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
