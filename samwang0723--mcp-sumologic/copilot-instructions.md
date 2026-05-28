## mcp-sumologic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP server for Sumo Logic log searches. Exposes a `search_sumologic` tool via the Model Context Protocol over Streamable HTTP transport (Express server on port 3006).

## Commands

```bash
npm install          # Install dependencies
npm run build        # Compile TypeScript (tsc + tsc-alias for path aliases)
npm start            # Run compiled server from dist/
npm run dev          # Dev mode with nodemon + tsx (auto-reload)
npm test             # Run Jest tests
npm run lint         # ESLint
npm run lint:fix     # ESLint with auto-fix
npm run format       # Prettier format
npm run format:check # Prettier check

# Docker
docker build -t mcp-sumologic .
docker run --rm --env-file .env -p 3006:3006 mcp-sumologic
docker-compose up --build -d
```

## Architecture

```
src/
├── index.ts                      # Express server + MCP setup (Streamable HTTP transport)
├── domains/sumologic/client.ts   # Search orchestration (job → poll → messages → cleanup)
├── lib/sumologic/
│   ├── client.ts                 # Sumo Logic HTTP client (Search Job API wrapper)
│   └── types.ts                  # TypeScript interfaces for Sumo Logic API
└── utils/pii.ts                  # PII masking (email, phone, CC, SSN, address)
```

### Request Flow

1. **MCP entry** (`index.ts`): Express receives MCP requests at `/mcp`, manages session-based transports. The `search_sumologic` tool accepts `query`, optional `from`/`to` ISO timestamps.
2. **Search orchestration** (`domains/sumologic/client.ts`): Creates a Sumo Logic search job, polls status until `DONE GATHERING RESULTS`, fetches messages, deletes the job. Default time range is last 24 hours, timezone `Asia/Hong_Kong`.
3. **HTTP client** (`lib/sumologic/client.ts`): Wraps `request-promise-native` with basic auth. Methods: `job()`, `status()`, `messages()`, `records()`, `delete()`.
4. **PII filtering** (`utils/pii.ts`): Applied only to `_raw` and `response` fields in search results. Redacts emails, credit cards, phone numbers, addresses, SSNs.

### Key Technical Details

- **ESM modules**: `"type": "module"` in package.json — all imports use `.js` extensions
- **Path aliases**: `@/*` maps to `src/*` (tsconfig paths + `tsc-alias` for build, `tsx` handles in dev)
- **Transport**: Streamable HTTP (not stdio) — each session gets its own `StreamableHTTPServerTransport` instance keyed by session ID
- **Health endpoint**: `GET /health` returns service status and enabled tools

## Environment Variables

Required in `.env`:
- `ENDPOINT` — Sumo Logic API base URL (e.g., `https://{host}/api/v1`)
- `SUMO_API_ID` — API access ID
- `SUMO_API_KEY` — API access key
- `PORT` — Server port (default: 3006)

---
> Source: [samwang0723/mcp-sumologic](https://github.com/samwang0723/mcp-sumologic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
