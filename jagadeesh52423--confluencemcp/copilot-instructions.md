## confluencemcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Running
```bash
# Build the TypeScript project
npm run build

# Start the MCP server (production)
npm start

# Start with auto-reload for development
npm run dev

# Clean build artifacts
npm run clean
```

### Configuration Setup
```bash
# Copy environment template
cp .env.example .env
# Then edit .env with your API credentials
```

## Architecture Overview

This is an **MCP (Model Context Protocol) server** that provides AI assistants with access to Atlassian APIs (Jira, Confluence, and Bitbucket). The server acts as a bridge between AI models and Atlassian services.

### Key Components

**Main Entry Point (`src/index.ts`):**
- Defines all available MCP tools (31 tools total)
- Handles tool routing and comprehensive error handling
- Registers service instances for each Atlassian product

**Configuration System (`src/config.ts`):**
- Supports service-specific API tokens (CONFLUENCE_*, JIRA_*, BITBUCKET_*)
- Falls back to legacy ATLASSIAN_* environment variables
- Validates configuration on startup

**Service Layer (`src/services/`):**
- `confluence.ts` - Confluence API operations (search, CRUD for pages/spaces)
- `jira.ts` - Jira API operations with **Smart Field Handling** for transitions
- `bitbucket.ts` - Bitbucket API operations (repos, PRs, issues, branches)

**Client Layer (`src/*-client.ts`):**
- Low-level HTTP clients with authentication and error handling
- One client per service (confluence, jira, bitbucket)

### Smart Jira Field Handling

The Jira service includes **Smart Field Handling** (`jira_transition_issue_interactive`) which:
- Analyzes issue content to suggest field values during transitions
- Recognizes patterns (DB scripts, test cases, templates)
- Provides contextual suggestions with reasoning
- Falls back gracefully when manual input is needed

### Configuration Architecture

- **Service-specific tokens** (recommended): `CONFLUENCE_API_TOKEN`, `JIRA_API_TOKEN`, `BITBUCKET_API_TOKEN`
- **Legacy fallback**: `ATLASSIAN_API_TOKEN` used when service-specific tokens are not provided
- **OAuth support**: Optional Jira OAuth via `JIRA_CLIENT_ID`/`JIRA_CLIENT_SECRET`

### Error Handling Strategy

All MCP tools return structured error responses with:
- **Error Details**: Raw API response for debugging
- **Request Context**: Parameters that were sent
- **Tips**: Actionable suggestions for common issues
- **Error Classification**: Uses `isError: true` flag for client handling

## Integration Notes

### Claude Desktop Setup
Add to `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "atlassian": {
      "command": "node",
      "args": ["/path/to/dist/index.js"]
    }
  }
}
```

### TypeScript Configuration
- Target: ES2022 with ESNext modules
- Outputs to `dist/` directory with source maps
- Strict type checking enabled

### Important Files
- `.env.example` - Template for all required environment variables
- `src/types.ts` - All TypeScript interfaces including Smart Field Handling types
- `src/services/jira.ts:50-200` - Smart field suggestion logic for transitions

---
> Source: [jagadeesh52423/ConfluenceMcp](https://github.com/jagadeesh52423/ConfluenceMcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
