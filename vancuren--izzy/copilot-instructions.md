## izzy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IzzyClaude is an always-on AI voice companion built with Next.js 16. It combines browser-based speech recognition/synthesis with a server-side agentic loop powered by Claude Sonnet 4. The agent can search the web, read URLs, remember user facts, reason step-by-step, and self-extend by building new capabilities in E2B sandboxes.

## Commands

- `npm run dev` — Start dev server with HTTPS (required for Web Speech API)
- `npm run build` — Production build
- `npm run lint` — ESLint (flat config, v9)
- `npm start` — Start production server

## Environment Variables

Copy `.env.example` to `.env.local`. Required keys:
- `ANTHROPIC_API_KEY` — Claude API
- `E2B_API_KEY` — E2B sandbox execution
- `TAVILY_API_KEY` — Web search
- `DB_PATH` — SQLite path (default: `data/izzy.db`)
- `IDLE_TIMEOUT_MS` / `NEXT_PUBLIC_IDLE_TIMEOUT_MS` — Idle auto-prompt timeout (default: 300000ms)

## Architecture

### Data Flow

```
Speech input (Web Speech API)
  → POST /api/chat { messages }
  → Memory decay + keyword extraction + context injection
  → runAgenticLoop (lib/tools/agentic-loop.ts)
    → Claude Sonnet 4 with tools (up to 10 rounds)
    → Tool executions emit SSE events (tool_start, tool_result)
  → SSE stream back to client
  → Text-to-speech output
```

### Client-Side State Machine

Defined in `lib/agent/state-machine.ts`. States: `idle` → `listening` → `thinking` → `speaking` → `idle`. The main React hook `lib/agent/use-agent.ts` manages speech recognition, SSE stream parsing, tool activity display, and builder subscriptions.

### Server-Side Agentic Loop

`lib/tools/agentic-loop.ts` — Calls Claude with assembled tools, executes tool_use blocks, feeds results back, repeats up to 10 rounds. Returns final text response plus any tool events emitted via SSE.

`app/api/chat/route.ts` — The chat endpoint. Decays memories, extracts keywords from recent messages, fetches relevant memories and active capabilities, builds the system prompt with context, runs the agentic loop, and fires off async memory extraction.

### Tool System

**Registry** (`lib/tools/registry.ts`) — Assembles built-in tools + active catalog capabilities into Anthropic tool format. Catalog capabilities are prefixed `cap_`.

**Executor** (`lib/tools/executor.ts`) — Routes tool calls by name. Built-in tools dispatch to handlers; `cap_*` names delegate to capability sandbox execution.

**ToolContext** (`lib/tools/context.ts`) — Injected into tool handlers with `messages`, `memory` (create/search), `catalog` (lookup/list/create), and `onToolEvent` callback.

**Built-in tools** (`lib/tools/built-in-tools.ts`): `lookup_capability`, `request_capability`, `execute_capability`, `ask_user`, `store_memory`, `recall_memory`, `web_search`, `browser_use`, `deep_memory`, `reason`.

**Tool primitives** live in `lib/tools/primitives/` (web-search, browser-use, deep-memory, reason).

### Capability System (Self-Extending Agent)

The agent can request new capabilities at runtime. Flow:

1. Agent calls `request_capability` tool → creates catalog entry (status: `building`)
2. Builder subagent (`lib/builder/agent-loop.ts`) spins up a 10-min E2B sandbox
3. Builder uses Claude to write Python, test it, and register the result
4. Progress streams to client via SQLite queue (`lib/queue/builder-queue.ts`) → SSE at `/api/builder/status?buildId=`
5. On success, capability status → `active`, becomes available as `cap_{name}` tool

**Capability convention**: Each capability is a Python file with `def run(args: dict) -> str:` that receives a dict and returns a conversational string. Stored in `data/capabilities/{uuid}/`.

### Memory System

SQLite-backed (`lib/memory/`). Memories have `tier` (short_term/long_term), `priority`, `decay_rate`, and `tags`. Search scores combine keyword matches, tag matches, content relevance, and recency. Decay runs periodically to prevent stale memories from dominating. Memory extraction from conversations is async (fire & forget) using Claude.

### Database

SQLite via `better-sqlite3` in WAL mode. Tables: `capabilities`, `memories`, `memory_edges`, `builder_queue`. Schema files in `lib/capabilities/schema.ts` and `lib/memory/schema.ts`.

## Key Conventions

- **Next.js App Router** with server-side API routes (no pages router)
- **Tailwind v4** (imported via `@import "tailwindcss"` in globals.css, no tailwind.config file)
- **TypeScript strict mode** with `@/*` path alias
- **Native modules** (`better-sqlite3`, `@e2b/code-interpreter`) are configured as server external packages in `next.config.ts`
- **SSE streaming** pattern: API routes return `ReadableStream` with `text/event-stream` content type; events defined in `lib/tools/stream-types.ts`
- **No test framework** is currently configured

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vancuren)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vancuren)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
