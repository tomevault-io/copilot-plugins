## mm2-blog-chat

> mm2-blog-chat — Cloudflare Worker providing AI chat backend for [mm2-blog](https://github.com/milkmaccya2/mm2-blog). RAG-based blog content search with Anthropic Claude streaming responses. Deployed at chat.milkmaccya.com.

# CLAUDE.md

## Project Overview

mm2-blog-chat — Cloudflare Worker providing AI chat backend for [mm2-blog](https://github.com/milkmaccya2/mm2-blog). RAG-based blog content search with Anthropic Claude streaming responses. Deployed at chat.milkmaccya.com.

## Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Dev server (localhost:8787) |
| `npm run deploy` | Deploy to Cloudflare Workers |
| `npm run lint` | Biome check |
| `npm run lint:fix` | Auto-fix Biome issues |

## Tech Stack

Cloudflare Workers, AI SDK (`@ai-sdk/anthropic`, `ai`), Cloudflare Vectorize (bge-m3), TypeScript, Biome, Lefthook

## Code Style (Biome)

- 2-space indent, 100 char line width, LF line endings
- Single quotes, semicolons always, ES5 trailing commas
- Pre-commit hook via Lefthook runs `npm run lint`

## Architecture

- `src/index.ts` — POST /chat endpoint with CORS
- `src/rag.ts` — Vectorize embedding search
- `src/system-prompt.ts` — LLM system prompt builder
- `src/constants.ts` — Shared embedding model constant
- `scripts/` — Ingest pipeline for Vectorize data

## CORS

Allowed origins: `https://blog.milkmaccya.com` and `*-mm2-blog.milkmaccya2.workers.dev` (PR previews).

## Environment Variables

`ANTHROPIC_API_KEY` — set via `wrangler secret put`

## Git Branching

Create topic branches from `main`. Never commit directly to `main`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/milkmaccya2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
