## ruwetmeter

> Build and run RuwetMeter, a public sentiment analysis system for Indonesia, across 5 phases. Currently **all phases complete** — backend, frontend, testing, critical fixes applied.

# RuwetMeter — Agent Guide

## Goal

Build and run RuwetMeter, a public sentiment analysis system for Indonesia, across 5 phases. Currently **all phases complete** — backend, frontend, testing, critical fixes applied.

## Commands

| Command                       | Description                                   |
| ----------------------------- | --------------------------------------------- |
| `bun run dev`                 | Start backend dev server with --watch         |
| `bun run typecheck`           | TypeScript type check (tsc --noEmit)          |
| `bun run check` (frontend)    | Svelte type check (svelte-check)              |
| `bun test`                    | All tests                                     |
| `bun test tests/unit/`        | Unit tests only                               |
| `bun test tests/integration/` | Integration tests only                        |
| `bun run db:generate`         | Generate Drizzle migration from schema        |
| `bun run db:migrate`          | Apply migration to database                   |
| `bun run db:push`             | Push schema directly (without migration file) |
| `bun run format`              | Prettier (no semi, single quote)              |
| `docker compose up -d db`     | Start PostgreSQL + pgvector locally           |

Dev order: `typecheck → check → test` (typecheck first).

## Architecture

### Dependency rule (strict, enforced by convention)

```
domain/ → application/ → infrastructure/  (one-way)
```

- `domain/`: zero dependencies
- `application/`: only imports from `domain/` and `ports/`
- `infrastructure/`: adapter implementations, may import from `application/` and `domain/`

### LLM integration

- **Provider-agnostic Strategy Pattern** via Provider Factory (Anthropic, OpenAI, Google, OpenRouter, OpenCode Zen, DeepSeek, Mistral, Groq).
- Embedding dimension fixed at 1536 (OpenAI `text-embedding-3-small` schema)

### Key decisions

| Aspect             | Choice                                                     |
| ------------------ | ---------------------------------------------------------- |
| Frontend scope     | SPA Dashboard + Chat                                       |
| LLM Architecture   | Provider Factory (Strategy Pattern)                        |
| Content extraction | Full article scraping via Readability                      |
| Embedding          | Agnostic (via EmbeddingProviderPort)                       |
| Request validation | Zod middleware (`@hono/zod-validator`)                     |
| Testing DB         | Testcontainers (Docker always available)                   |
| Runtime            | Bun 1.3.3 + Hono 4.12                                      |
| ORM                | Drizzle 0.40 + PostgreSQL + pgvector                       |
| Frontend framework | SvelteKit + Tailwind v4 + daisyUI v5                       |
| Fonts              | Playfair Display (heading), Plus Jakarta Sans (body)       |
| Language           | Indonesian for UI/LLM, English for code                    |
| Code style         | Prettier: no semi, single quote, trailing comma, 100 width |

## Progress

### Done

- **Fase 1–5 complete:** full project scaffold, Clean Architecture, all layers, frontend SPA, testing.
- **4 critical fixes:** transaction leak (removed fake BEGIN/COMMIT), DOMParser crash (fast-xml-parser), LLM hallucination (clamp), embedding rate limit (chunked batches).
- **PostgreSQL:** vector(N) parameterization, history endpoint (inArray), pgvector auto-enable (init.sql), HNSW index.
- **RSS:** 8 working Indonesian feeds (Kontan removed — unreachable).
- **Providers:** OpenCode Zen (analysis + chat), OpenRouter (gpt-4o-mini), DeepSeek/Mistral/Groq stubs.
- **Frontend:** Tailwind v4 + daisyUI v5 CSS-first config, fonts via @fontsource, dark mode (localStorage), FAB chat, messenger bubbles, glass-morphism, dashboard with hero score + 4-col dimensions + summary + chart.
- **Headings:** All labels use `font-heading` (Playfair Display): "Indeks Ruwet", "Ringkasan", "Tren Historis", "Ekonomi", "Politik", "Infrastruktur", "Sosial".
- **Indonesia map:** `IndonesiaMap.svelte` component — fetch SVG from `/indonesia-map.svg`, inline via `@html`, `preserveAspectRatio="xMidYMid slice"`. Fill colors replaced via props (`color`, `secondaryColor`) → `rgba()` per theme. Reactive via `$effect` on theme toggle.
- **lucide-svelte:** v1.0.1, imports via subpath `lucide-svelte/icons/icon-name` (not barrel). Config: `ssr.noExternal` + `optimizeDeps.exclude` in `vite.config.ts`.
- **Calm palette:** All vibrant colors replaced — score bars use `base-content/20`, FAB/send buttons `base-content/10`, chart original (threshold: ≥70 → red).
- **Score colors:** Total + dimension scores turn `text-red-500/80` at ≥70; progress bar fill becomes `bg-red-500/40`.
- **Chart:** Label format `dd Mon HH:00` (data updates every 3h).
- **Typecheck:** `bun run check` = 0 errors (frontend).
- **Git:** 3 commits.
- **RMS (Root Mean Square) for Total Score:** Replaced simple average calculation to prevent score dilution.
- **Per-dimension Time-Weighted RAG Decay:** Dynamic decay half-life query in DB (`CASE WHEN score > 50 THEN 3/2 days ELSE default`) utilizing cleaner options object structure in use-case and repository.
- **DB Schema:** Updated `news_articles` to store `score_economy`, `score_politics`, `score_infrastructure`, and `score_social` (migration `0003_melted_polaris` applied).
- **LLM Adapters:** Updated working adapters (`OpenAIAnalysisAdapter`, `GoogleAnalysisAdapter`, `AnthropicAnalysisAdapter`) to prompt and parse individual `articleScores` for database persistence.

### Blocked

- OpenRouter 402 error (insufficient credits for full article payload). Workaround: OpenCode Zen for analysis.
- DeepSeek, Mistral, Groq adapters throw "not implemented".
- 2 repository integration tests (testcontainers) fail — container startup timeout.

## DB schema (implemented)

4 tables: `news_articles`, `ruwet_logs`, `ruwet_log_articles` (junction), plus `content_type` enum.

- `news_articles.embedding` = `vector(1536)` for pgvector
- UNIQUE constraint on `url` (RSS deduplication at DB level)
- B-Tree indexes on `published_at` and `created_at`
- Custom CHECK constraints + HNSW index registered in `meta/_journal.json`

## Env vars

See `.env.example`. All required unless a default exists:

- `DATABASE_URL` — required
- `ANALYSIS_PROVIDER`, `CHAT_PROVIDER`, `EMBEDDING_PROVIDER` — required
- `RSS_FEED_URLS` — comma-separated
- Validation in `src/config.ts` — fails at startup if missing

## Important file locations

- `frontend/src/app.css`: Tailwind v4 CSS-first config + daisyUI + custom animations
- `frontend/src/routes/+layout.svelte`: map background img, navbar, FAB chat button
- `frontend/src/routes/+page.svelte`: dashboard (hero score, 4 dimensions, summary, chart)
- `frontend/src/routes/chat/+page.svelte`: chat page
- `frontend/src/lib/components/ChatBox.svelte`: messenger-style bubbles
- `frontend/static/indonesia-map.svg`: cleaned SVG with transform group
- `frontend/vite.config.ts`: @tailwindcss/vite + sveltekit
- `frontend/package.json`: tailwindcss v4.3, daisyui v5.6, @fontsource/playfair-display, plus-jakarta-sans
- `backend/src/config.ts`: env validation, 8-provider key map
- `backend/src/infrastructure/llm/provider-factory.ts`: switch-case factory
- `backend/src/infrastructure/llm/adapters/openai.adapter.ts`: full OpenAI-compatible adapter
- `backend/src/infrastructure/llm/adapters/openrouter.adapter.ts`: delegates to OpenAI adapter
- `backend/src/infrastructure/llm/adapters/opencode.adapter.ts`: delegates to OpenAI adapter

## Notes for agents

- All code in English, UI/LLM text in Indonesian
- `ScoreDimension` constructor: validates integer 0–100, throws if invalid
- Total score = Root Mean Square (RMS) of 4 dimensions; flagged if delta > 30 from previous cycle
- LLM adapters implement AnalysisProviderPort, ChatProviderPort, EmbeddingProviderPort
- Chat single-turn; session_id accepted but not processed yet
- Telegram webhook validates X-Telegram-Bot-Api-Secret-Token header
- Rate limiter in-memory (Map), not shared across instances
- Data retention: news_articles > 90 days deleted (scheduled job not implemented yet)
- Time-weighted RAG: cosine similarity × exponential decay with dynamic per-dimension half-life (politics: 3 days if > 50, others: 2 days if > 50, fallback to default), in SQL via `semanticSearch`
- Don't write too many comments

---
> Source: [ikhsan3adi/RuwetMeter](https://github.com/ikhsan3adi/RuwetMeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
