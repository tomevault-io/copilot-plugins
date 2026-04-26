## debate-arena

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Debate Arena — a React web app that orchestrates real-time debates between three AI agents (Advocate/Google Gemini, Critic/OpenAI, Wildcard/Anthropic Claude) across 3 rounds on a user-submitted topic. The Wildcard judges each round. Results are visualized as a D3 force-directed graph.

## Commands

```bash
npm run dev        # Vite dev server (http://localhost:5173)
node server.js     # Express API server (http://localhost:3001)
npm run build      # Production build → dist/
npm run lint       # ESLint
npm start          # Production: serves dist/ + API
```

Development requires both the Vite dev server and the Express backend running. Vite proxies `/api/*` to localhost:3001.

No test suite is configured.

## Architecture

**Frontend**: React 19 + Vite 8, no TypeScript, no state management library. All state lives in App.jsx via hooks.

**Backend**: `server.js` — Express server with multi-provider LLM calls (Anthropic, OpenAI, Google) and in-memory rate limiting. Two endpoints: `POST /api/debate` and `POST /api/verdict`.

**Data flow**: TopicInput → `runDebate()` (src/lib/debate.js) loops rounds×agents → each call hits `/api/debate` → response parsed into claim objects → `buildGraphData()` (src/lib/graphUtils.js) generates nodes/links → DebateGraph renders via D3.

**Key modules**:
- `src/lib/agents.js` — Agent config (names, colors, prefixes) + response parsers. Claim IDs follow `{prefix}_r{round}_{index}` pattern (e.g., `adv_r1_1`).
- `src/lib/debate.js` — Async debate orchestrator with abort support.
- `src/lib/graphUtils.js` — Graph data builder, scoring (`computeWildcardScore`), round winner logic.
- `src/components/DebateGraph.jsx` — D3 SVG force graph (800×700 viewbox). Agent anchors: Advocate top-center, Critic bottom-left, Wildcard bottom-right.

**Agent colors**: Green (Advocate), Red (Critic), Purple (Wildcard).

## Environment Variables

Set in `.env.local`: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GOOGLE_API_KEY`. Optional model/token overrides per provider (e.g., `ANTHROPIC_MODEL`, `OPENAI_MAX_TOKENS`). Rate limit config: `RATE_LIMIT_IP_DAILY`, `RATE_LIMIT_GLOBAL_DAILY`, `DEBATE_COOLDOWN_MS`.

## Notes

- JSON parsing in agents.js is lenient — strips markdown code fences and extracts JSON from mixed LLM text.
- Deployed at `https://debate-arena-ten.vercel.app`. No CORS config needed — API functions are same-origin (Vercel serverless).
- Rate limiting is in-memory; resets on server restart.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/franalli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
