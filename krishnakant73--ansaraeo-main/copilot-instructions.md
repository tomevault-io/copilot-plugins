## ansaraeo-main

> Guidance for Claude Code in this repo. Keep edits to this file lean — long batch history belongs in `BATCH*_SETUP_NOTES.md`, not here.

# CLAUDE.md

Guidance for Claude Code in this repo. Keep edits to this file lean — long batch history belongs in `BATCH*_SETUP_NOTES.md`, not here.

## What this app is
**AnsarAEO** — AI Search / Answer Engine Optimization (AEO) SaaS for Indian brands. Tracks brand *mentions* across AI answer engines (ChatGPT, Perplexity, Gemini, Google AI Overviews, Copilot, Grok), scores visibility, audits sites for AI-citability, generates draft content, competitor intelligence, local SEO, revenue attribution. INR pricing, multilingual + WhatsApp.

Stack: **Next.js 16 (App Router) + TypeScript**, **Supabase** (Postgres + Auth + RLS), **Tailwind**, **@react-pdf/renderer**, **vitest**, **Vercel** (cron in `vercel.json`).

## Commands
- `npm run dev` / `build` (runs type-check) / `start` / `lint` / `test` (vitest run) / `typecheck` (tsc --noEmit) / `test:ci`
- Single test: `npx vitest run src/lib/utils.test.ts`

## Domain model (mental model)
orgs → **brands** → **prompts** (questions to be mentioned for, language-tagged) → **visibility_runs** (one prompt × each active engine; stores raw_response, brand_mentioned, citations, competitor_mentions JSONB, mention_verification JSONB). Plus: competitors, content_items, integrations, site_audits, payments/plan_limits, agent_conversations, brand_positioning/brand_perception.

## Core pipeline — `src/lib/visibility-engine.ts`
`runVisibilityCheck(promptId)`: loads prompt+brand+competitors+active engines (service client) → per engine calls `ENGINE_CALLERS` registry (`Record<engineName, callerFn>`; add engines here, keep name in sync with `engines.name`) → `gpt-4o-mini` JSON-mode classification → **reconciles** against deterministic `src/lib/mention-matcher.ts` (deterministic wins for literal name presence; LLM stays authoritative for sentiment/position; logged in `mention_verification`, not silently resolved) → inserts `visibility_runs` + `citations`. Invoked by `POST /api/visibility-check`, `/api/cron/nightly-runs`, `/api/content/generate`.
`google_ai_overview` ≠ `gemini`: it scrapes real AI Overview boxes via DataForSEO (`src/lib/google-ai-overview.ts`) and **skips** (no "0%") when a query triggers no AI Overview.

## Supabase clients — use the right one
- `createClient()` (`src/lib/supabase/server.ts`) — cookie client, **respects RLS**, user-scoped. Use for any user-facing query.
- `createClient()` (`src/lib/supabase/client.ts`) — browser client.
- `createServiceClient()` — **service-role, BYPASSES RLS**. Server-only trusted work (cron, reports, visibility runs). Never in client components; never to browser.

## Auth / brand selection
- `handle_new_user()` trigger auto-creates org + org_member on signup. RLS funnels every query through `org_members → brands → …`.
- `selected_brand_id` cookie → `getSelectedBrand()` (`src/lib/selected-brand.ts`), fallback to first brand.
- `src/proxy.ts` refreshes session + redirects unauth from `/dashboard/:path*`. `dashboard/layout.tsx` also `redirect("/login")` if no user.

## Modules (source of truth = code; condensed)
- `content-engine.ts` Content Studio (draft, `[ADD …]` placeholders) · `content-optimizer.ts` · `pdp-generator.ts` (evidence ledger) · `answer-blocks.ts` · `fanout-coverage.ts` · `geo-linter.ts` (deterministic, no LLM/key) · `llms-txt-validator.ts` (deterministic) · `robots-validator.ts` (spec-accurate) · `schema-for-ai.ts` (+ `validateJsonLd`) · `site-audit-engine.ts` · `ai-index-generator.ts` (llms.txt/robots/JSON-LD/aeo.json/entity.json, IndexNow) · `internal-link-graph.ts` · `header-link-graph.ts` · `topical-coverage.ts` · `token-bloat.ts` · `price-factcheck.ts` · `blind-discovery.ts` / `visibility-consistency.ts` (export `callEngine`) · `competitor-intel.ts` · `content-gap.ts` · `social-signals.ts` (Reddit/YouTube) · `brand-perception.ts` (+ `-io.ts`) · `gbp-audit.ts` · `gsc.ts` (OAuth) · `revenue-attribution.ts` · `crypto.ts` (AES-256-GCM) · `whatsapp.ts` · `razorpay.ts` (`getRazorpay()`) · `reports.ts` + `report-document.tsx` (shared PDF) · `agent-context.ts` (`/api/agent/chat`).
- Routes: `(auth)/`, `(marketing)/`, `dashboard/*`, `api/*`. Cron routes (`/api/cron/*`, `/api/whatsapp/send-digest`) gated by `CRON_SECRET` (`Bearer ${CRON_SECRET}`).
- `cn()` from `src/lib/utils.ts` — Tailwind class-merge, used everywhere.

## STRICT style / safety rules (do not skip)
- **HONESTY DESIGN:** generation is always a DRAFT with `[ADD …]` placeholders for owner-only facts; never auto-fill invented specifics. Stateless/generation-only features persist nothing — do NOT fabricate report/PDF sections for them (only surface features with a persisted table).
- **Razorpay MUST be lazy-init** via `getRazorpay()` inside the request handler — never a top-level `new Razorpay(...)` (breaks `next build` when secrets absent).
- **Mention classification: deterministic over LLM self-report** for literal name presence.
- **Gemini ≠ Google AI Overview** — keep separate.
- **Per-engine failure isolation** via `Promise.allSettled`; `grok` skips without `GROK_API_KEY`, `copilot` skips without `COPILOT_API_URL`/`COPILOT_API_KEY`. Never fake an engine API.
- **Shared report code path:** add report fields in `reports.ts`, not duplicated route PDF logic.
- **Migrations sequential:** `schema.sql` → `migration_002`…`011` (+ `migration_015_brand_positioning`) in order. `supabase/reset.sql` wipes+reseeds.
- **RLS org-scoped on all tables** — user-facing code uses the cookie client.
- **`ENCRYPTION_KEY`** (32-byte hex, AES-256-GCM) for integration creds — never plaintext, never committed, never logged.
- **Razorpay webhook** (`/api/billing/webhook`) MUST verify signature before trusting payload.
- **Tests:** vitest, scoped `src/**/*.test.ts`, **relative imports only** (`./utils`, no `@/` — no vitest alias). Deterministic parsers tested with `vi.stubGlobal("fetch", …)`.

## Env vars (see `.env.all.example`)
Required: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `OPENAI_API_KEY`, `PERPLEXITY_API_KEY`, `GOOGLE_AI_API_KEY`, `CRON_SECRET`, Razorpay (`RAZORPAY_KEY_ID/SECRET/WEBHOOK_SECRET`), WhatsApp (`WHATSAPP_ACCESS_TOKEN/PHONE_NUMBER_ID/WEBHOOK_VERIFY_TOKEN`), `ENCRYPTION_KEY` (32-byte hex), Zoho SMTP. Optional: `GROK_API_KEY`, `COPILOT_API_URL/KEY/MODEL`, `GOOGLE_PLACES_API_KEY` (GBP), `YOUTUBE_API_KEY`, `GOOGLE_CLIENT_ID/SECRET` (GSC), `DATAFORSEO_LOGIN/PASSWORD` (google_ai_overview), `MONITORING_WEBHOOK_URL`. GA4/Shopify creds entered per-brand in UI, not env vars.

## Reference docs (in repo)
`BATCH*_SETUP_NOTES.md` (per-batch history), `BUILD_INDEX.md`, `PRD.md` (full feature catalog + UI breakdown), `AUTH_DEBUGGING_AND_GOOGLE_SETUP.md`, `INTEGRATION_GUIDE.md`, `PRODUCTION_DEPLOYMENT_CHECKLIST.md`, `FINAL_GO_LIVE_CHECKLIST.md`. (External `0x-*.md` planning docs are NOT in this repo.)

## gstack
Skill suite at `.claude/skills/gstack`. For web browsing use `/browse`; **never `mcp__claude-in-chrome__*`**.

---
> Source: [Krishnakant73/ansaraeo-main](https://github.com/Krishnakant73/ansaraeo-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
