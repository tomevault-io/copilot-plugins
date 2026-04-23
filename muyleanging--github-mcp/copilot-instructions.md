## github-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
npm install

# Run integration tests (macOS/Linux)
GITHUB_TOKEN=ghp_yourtoken node client.js

# Run integration tests (Windows PowerShell)
$env:GITHUB_TOKEN="ghp_yourtoken"; node client.js

# Start MCP server directly
GITHUB_TOKEN=ghp_yourtoken node server.js
```

`GITHUB_TOKEN` is required for both `server.js` and `client.js`. The token needs scopes: `repo`, `read:org`, `read:user`.

## Architecture

This is a GitHub MCP (Model Context Protocol) Server — a stdio-based JSON-RPC server that exposes GitHub API operations as tools that Claude Desktop can call.

**Data flow:**
```
Claude Desktop → MCP protocol (JSON-RPC/stdio) → server.js → api.github.com
```

### Key Files

- **`server.js`** — The MCP server. Registers 8 GitHub tools and 2 resources, then handles tool call requests by making HTTPS requests to `api.github.com`. Uses `StdioServerTransport` so it communicates via stdin/stdout (no network port).
- **`client.js`** — Integration test client. Spawns `server.js` as a child process, performs an MCP handshake, then sequentially calls each tool and reports pass/fail results.

### Tools Exposed by server.js

`get_user`, `list_repos`, `get_repo`, `list_issues`, `list_pull_requests`, `get_file_contents`, `search_repos`, `list_commits`

Resources: `github://user/profile`, `github://user/repos`

### MCP Protocol

The server uses `@modelcontextprotocol/sdk` (single dependency). Tools are defined declaratively with JSON Schema input descriptions, then implemented via `server.setRequestHandler(CallToolRequestSchema, ...)`. The GitHub API helper at the top of `server.js` handles authentication and HTTPS requests.

## Claude Desktop Integration

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "github": {
      "command": "node",
      "args": ["/full/path/to/server.js"],
      "env": {
        "GITHUB_TOKEN": "ghp_yourtoken"
      }
    }
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MuyleangIng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
