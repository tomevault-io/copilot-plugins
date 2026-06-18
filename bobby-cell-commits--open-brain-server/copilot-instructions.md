## open-brain-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open Brain is a self-hostable MCP memory server for AI assistants. Thoughts are captured from Telegram (or any MCP client), embedded with vector search via OpenRouter, stored in Supabase with pgvector, and exposed through an MCP server so any AI assistant can search/write to the same shared memory.

## Architecture

**Capture path:** Source → Edge Function → OpenRouter (embedding + metadata extraction in parallel) → Supabase insert → auto-link similar thoughts → extract entities

**Retrieval path:** MCP client call → Edge Function → tool execution → JSON response

**Storage:** Supabase Postgres + pgvector. `thoughts` table with 1536-dim embeddings, JSONB metadata, idempotency via `(source, source_event_id)` partial unique index. RPC `match_thoughts` for similarity search.

**Auth:** MCP key via `x-brain-key` header. Multi-tenant — each API key maps to an isolated brain.

## Code Layout

- `supabase/functions/` — Deno-based Supabase Edge Functions
  - `_shared/` — Shared modules: `supabase-client.ts`, `openrouter.ts`, `types.ts`, `errors.ts`
  - `telegram-bot/` — Telegram webhook handler (primary capture path)
  - `open-brain-mcp/` — MCP server (14 tools). See `docs/cookbook.md` for usage guide.
  - `run-pipeline/` — Edge Function for automated RSS, HF Papers, and Emergent Mind ingestion
  - `monitor-pipeline/` — Pipeline health monitoring with Telegram alerting
- `supabase/migrations/` — Ordered SQL migrations applied via Supabase CLI
- `pipeline/` — Python-based pipeline scripts
  - `config.py` — Env loading, subreddit list, feed URLs, focus terms
  - `openbrain_client.py` — JSON-RPC client for capture_thought
  - `triage.py` — LLM triage via OpenRouter
  - `dedup.py` — JSON-file-based processed item tracker
  - `reddit/` — Subreddit monitoring (public .json endpoints)
  - `rss/` — RSS/Atom feed poller
  - `emergentmind/` — Emergent Mind arXiv trending papers
  - `hf_papers/` — HuggingFace daily papers
  - `briefing/` — Morning briefing generator
- `scripts/` — Bootstrap, deploy, validate scripts
- `tests/integration/` — Multi-tenant integration tests
- `docs/` — Cookbook, source-writing guide, tracker guide, skill showcases

## Runtime & Language

- **Edge Functions:** Deno + TypeScript, `npm:` specifiers, no package.json
- **Pipeline:** Python 3.13+, dependencies in `pipeline/requirements.txt`
- Run pipeline as module from repo root: `python -m pipeline.run_all`

## Key Environment Variables

**Edge Functions** (in `supabase/.env.local`): `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `OPENROUTER_API_KEY`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_SECRET_TOKEN`, `TELEGRAM_ALLOWED_CHAT_ID`, `MCP_ACCESS_KEY`

**Pipeline** (in `pipeline/.env`): `OPENBRAIN_MCP_URL`, `OPENBRAIN_KEY`, `OPENROUTER_API_KEY`

## Conventions

- Edge Functions return JSON responses with `Content-Type: application/json`
- Errors use `errorResponse(message, status)` from `_shared/errors.ts`
- Idempotency: Telegram captures use `source='telegram'` + `message_id`; MCP captures use `source='mcp'` + `null` event ID; pipeline captures use `source='reddit'|'rss'` + item-specific event ID
- Embedding dimension is locked at 1536 (defined in `types.ts`)
- All OpenRouter calls go through `_shared/openrouter.ts` — never call the API directly
- Row Level Security enabled on all tables — direct database access via anon key is blocked

## Adding a Pipeline Source

See `docs/writing-a-source.md` for the fetch→dedup→triage→capture pattern and a minimal skeleton. The RSS source (`pipeline/rss/fetcher.py`) is the cleanest reference implementation.

## MCP Tool Reference

See `docs/cookbook.md` for comprehensive usage patterns, composition recipes, and non-obvious behaviors for all 14 MCP tools.

---
> Source: [Bobby-cell-commits/open-brain-server](https://github.com/Bobby-cell-commits/open-brain-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
