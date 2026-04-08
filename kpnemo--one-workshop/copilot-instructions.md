## one-workshop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI-powered web scraper service. Takes multiple URLs + a topic, scrapes pages with Playwright, uses a Claude tool-use agent to extract topic-relevant information, and streams live status to a React frontend via SSE.

## Commands

### Development
```bash
pm2 start ecosystem.config.cjs    # Start both server (3001) and client (5173)
pm2 logs                          # Tail logs
pm2 restart all                   # Restart after changes
pm2 stop all                      # Stop everything
npm run dev                       # Alternative: concurrently runs both without pm2
```

### Testing (server only)
```bash
cd server && npm test                              # Run all tests
cd server && npx vitest run tests/tools.test.ts    # Run specific test file
cd server && npm run test:watch                    # Watch mode
```

### Building
```bash
npm run build    # Builds server (tsc) then client (vite)
```

## Architecture

**Monorepo** with `client/` (React + Vite) and `server/` (Express + TypeScript). No shared packages — types are duplicated between client and server and must be kept in sync manually.

### Data Flow
```
Client POST /api/scrape {urls[], topic}
  → Express validates, sets SSE headers
  → Playwright fetches all URLs in parallel (domcontentloaded, 30s timeout)
    ← SSE: fetching started/done/failed per URL
  → Agent extractor runs Claude tool-use loop (max 20 turns)
    ← SSE: thinking, tool_call, tool_result per turn
  → Agent calls submit_result with final JSON
    ← SSE: result event with extracted data
```

### SSE Streaming
The POST endpoint returns an SSE stream (not a JSON response). The client parses it with `fetch` + `ReadableStream` — `EventSource` API doesn't support POST. Three event types: `status`, `error`, `result`.

### Agent Tools (6 total)
- `search_content` — substring search across page text
- `extract_structured_data` — regex HTML table/list extraction
- `classify_relevance` — keyword density heuristic
- `extract_links` — find links on a page, optionally filtered
- `follow_link` — fetch a new URL mid-session (max 3 per session, launches new browser)
- `submit_result` — terminates agent loop with final JSON

The agent operates on a mutable page pool. `follow_link` adds pages to it, making them available to all other tools.

### Key Type Contracts
`server/src/types.ts` and `client/src/lib/types.ts` define the SSE event protocol. The `StatusEvent` discriminated union (phase + state) drives the entire streaming UI. Changes to these types must be synchronized across both files.

## Environment
- `ANTHROPIC_API_KEY` in root `.env` (loaded by server via dotenv)
- Server loads `.env` from `path.resolve(__dirname, "../../.env")` relative to `server/src/`
- Vite proxies `/api` to `http://localhost:3001` in dev

## Model
Agent uses `claude-sonnet-4-20250514`. Configured in `server/src/agent/extractor.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kpnemo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kpnemo)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
