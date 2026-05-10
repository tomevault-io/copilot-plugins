## toodledo-mcp

> MCP server connecting Claude to Toodledo.com for GTD-based task management.

# Toodledo MCP Server

## Project Overview
MCP server connecting Claude to Toodledo.com for GTD-based task management.

## Tech Stack
- TypeScript, Node 24, ES modules
- `@modelcontextprotocol/sdk` for MCP server
- Native `fetch` for HTTP, native `http` for OAuth callback
- STDIO transport

## Build & Run
- `npm run build` - compile TypeScript to dist/
- `npm run dev` - run with tsx (development)
- `npm run setup` - one-time OAuth2 setup flow

## Architecture
- `src/auth/` - OAuth2 flow, token storage at ~/.toodledo-mcp/tokens.json
- `src/api/` - Toodledo API client with name-to-ID resolution cache
- `src/tools/` - MCP tool definitions (task CRUD, folder/context/goal mgmt, GTD queries)
- `src/index.ts` - server entry point

## Toodledo API
- Base URL: https://api.toodledo.com/3/
- Auth: OAuth2 with access_token parameter on all requests
- Tasks response has metadata object as first array element, skip it
- Status: 0=None, 1=Next Action, 2=Active, 3=Planning, 4=Delegated, 5=Waiting, 6=Hold, 7=Postponed, 8=Someday, 9=Canceled, 10=Reference
- Priority: -1=Negative, 0=Low, 1=Medium, 2=High, 3=Top

## User's GTD Workflow
- **Inbox = `.inbox` context** — NOT a folder. All new brain-dump tasks go here.
- Inbox is processed once daily into the GTD system (assigned folder, context, status, etc.)
- **Folders = Projects** or long-term collections (e.g. Ideas, Books to Read)
- **Contexts = Batching** for GTD context batching (e.g. @Office, @Home, @Gemba)
- When asked to show "inbox", search by context `.inbox`
- When adding a quick task / brain dump, set context to `.inbox` unless told otherwise

---
> Source: [wwilson1017/toodledo-mcp](https://github.com/wwilson1017/toodledo-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
