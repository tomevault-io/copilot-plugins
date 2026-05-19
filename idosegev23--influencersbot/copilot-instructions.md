## influencersbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
npm run dev          # Start dev server (with large header size for session cookies)
npm run build        # Production build
npm run type-check   # TypeScript check without emitting
npm run lint         # ESLint
npm run check:env    # Validate required environment variables

# Testing
npm run test             # Run unit tests (Vitest)
npm run test:ui          # Vitest with browser UI
npm run test:coverage    # Unit tests with coverage report
npm run test:e2e         # Playwright E2E (requires running dev server)
npm run test:e2e:ui      # Playwright UI mode

# Run a single test file
npx vitest run tests/unit/path/to/test.ts

# Database backups
npm run backup:db
npm run backup:all
```

## Architecture Overview

This is a **multi-tenant SaaS platform** for influencer management with AI-powered chatbots, document parsing, and Instagram analytics.

### Tech Stack
- **Next.js 16** (App Router) + TypeScript + Tailwind CSS 4
- **Supabase** (PostgreSQL + Auth + RLS) — multi-tenant with Row-Level Security
- **Upstash Redis** — caching and rate limiting
- **OpenAI / Google Gemini / Claude** — AI processing with multi-model fallback
- **Apify** — Instagram scraping

### Role System
4-tier RBAC: **Admin → Agent → Influencer → Follower**. Enforced via Supabase RLS policies. The `SUPABASE_SERVICE_ROLE_KEY` bypasses RLS for server-side admin operations.

### Message Processing Pipeline (`/src/engines/`)
Incoming chat messages flow through:
1. **Understanding engine** (`engines/understanding/`) — extracts intent, entities, risk flags via AI
2. **Decision engine** (`engines/decision/`) — routes to handler, applies cost/security/personalization rules
3. **Policy engine** (`engines/policy/`) — rate limiting and security checks
4. **Action** — chatbot response, task creation, etc.

The main orchestrator is `engines/index.ts`.

### Hybrid Retrieval Chat System (`/src/lib/chatbot/`)
The primary chatbot uses a **4-stage multi-retrieval approach** (see `HYBRID_RETRIEVAL.md`):
1. Full-text indexed search (2K token budget) across posts, transcriptions, highlights
2. AI selects what additional content is needed (function calling)
3. Fetch only the requested items
4. Generate final response

Key files: `sandwich-bot-hybrid.ts` (orchestrator), `hybrid-retrieval.ts` (retrieval logic).

### AI Document Parser (`/src/lib/ai-parser/`)
Multi-model fallback chain: **Gemini Flash → Claude → GPT-4o** for parsing partnership agreements, invoices, briefs. Supports Hebrew, English, Arabic, Russian. Confidence scoring with validation.

### Instagram Scraping (`/src/lib/scraping/`)
Orchestrated via `influencer-scrape-orchestrator.ts`. Apify actors fetch Instagram data; `newScanOrchestrator.ts` handles first-time scans, `incrementalScanOrchestrator.ts` handles updates.

### API Routes (`/src/app/api/`)
Key route groups:
- `/api/chat/*` — chat, hybrid retrieval, streaming, session management
- `/api/influencer/*` — auth, chatbot persona, documents, partnerships, coupons
- `/api/scraping/*` — start/status/step/cancel/retry scraping jobs
- `/api/admin/*` — admin operations

### Rate Limiting (`middleware.ts`)
Applied globally via Upstash Redis:
- `/api/chat`: 100 req/min
- `/api/auth`: 50 req/min
- `/api/influencer`: 200 req/min
- Admin routes: 20 req/min

### Path Aliases
`@/*` maps to `./src/*` — use for all internal imports.

### TypeScript
`strict: false` in tsconfig. Build errors are ignored in `next.config.ts` (`typescript.ignoreBuildErrors: true`), so always run `npm run type-check` separately.

### Project Documentation
`/memory-bank/` contains living documentation: `systemPatterns.md` (architecture decisions), `techContext.md` (DB schema detail), `activeContext.md` (current status), `progress.md` (milestones).

---
> Source: [Idosegev23/influencersBot](https://github.com/Idosegev23/influencersBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
