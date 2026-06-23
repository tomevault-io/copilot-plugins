## public-companies-data-scraper

> > Root context file. Re-injected every turn. Keep it lean — every token here is a recurring tax.

# CLAUDE.md — AI PE Deal Screener (Capstone Build)

> Root context file. Re-injected every turn. Keep it lean — every token here is a recurring tax.
> Single source of truth for system-wide invariants, architecture map, UX spec, and build sequence.
> Module detail lives in each subdirectory's CLAUDE.md. Contracts live in `/specs` and are FROZEN.
> Hard code lives in `/reference` — COPY it, do not reinvent it (see §6).

---

## READ ORDER (token-efficiency rule — enforced every session)

1. Read THIS file first, every session. It is the Map.
2. Then read ONLY the `CLAUDE.md` of the ONE module relevant to the current task (`/backend`, `/mcp-server`, or `/frontend`). Each module CLAUDE.md contains a **routing table** telling you exactly which files to READ, which to SKIP, and which tools to load per task type. Obey it.
3. NEVER load all workspaces, the full `/specs` set, or another module's internals. The HTTP/MCP contracts in `/specs` are the only cross-module interface — if you think you need another module's source, you're wrong; read its contract instead.

---

## 0. What this is (scope — do not exceed)

An **autonomous AI Private Equity Deal Screener**, built as a **portfolio capstone** (with possible small-scale use by known users — not a SaaS focus).

Full agentic system: niche keyword → candidate discovery (SEC EDGAR) → MCP-served financial reading → LLM screening against criteria → ranked report → Stripe-gated full results → Telegram alerts.

**Product model (BuiltWith pattern):** one free lookup with partial results, full results + monitoring behind Stripe. Demo data source is **SEC EDGAR only** (public, free, legal). Private-source connectors are documented as future extensions, NOT built.

**Out of scope for v1:** private deal-database integrations, multi-tenant orgs, reselling scraped proprietary data, trading functionality.

---

## 1. Non-negotiable invariants

- **Human-in-the-loop.** Output is a first-pass research filter, never a buy/sell decision.
- **Never assert unverified financials.** Every figure carries `verified: false` + `source_url`; UI shows the source link beside every number.
- **Contracts in `/specs` are frozen.** No changes without explicit instruction + a dated entry in `DECISIONS.md`.
- **No secrets in code or git.** Env vars only; ship `.env.example` with placeholders.
- **No raw HTML/JSON dumps to the LLM.** Deterministic parsing first; the model gets only distilled structured data.
- **Stripe = hosted Checkout + Customer Portal only.** No custom card forms. Entitlement driven by verified webhooks only.
- **Freemium gating is SERVER-SIDE.** The API truncates/blurs data for free users before it leaves the backend. Never send full data to the client and hide it with CSS.
- **SEC compliance:** every EDGAR request sends the declared `User-Agent` with contact email; global rate limit ≤ 8 req/s. (Implemented in `/reference/edgar_client.py` — use it.)

---

## 2. Architecture map

```
[Next.js frontend] --HTTP--> [FastAPI orchestrator] --calls--> [Screening Agent (LLM)]
                                      |                                |
                                      |--> [Source layer: EDGAR client] (discovery: full-text search + SIC)
                                      |--> [MCP server] (companyfacts XBRL -> FinancialSnapshot)
                                      |--> [Stripe] (Checkout + webhook entitlement)
                                      |--> [Telegram alerter]
                                      |--> [Postgres] (users, screens, candidates, results, entitlements, stripe_events)
                                      |--> [Async jobs] (screens write results row-by-row as they finish)
```

Components communicate ONLY through `/specs` contracts. No component imports another's internals.

---

## 3. WAT framework (full version in `/docs/WAT.md`)

**Workflows:**
- `run_screen`: keyword/criteria → EDGAR discovery → financials via MCP → agent screens each candidate (concurrent, bounded) → results persist **one row at a time as they complete** → rank → Telegram alert on completion.
- `recurring_monitor`: scheduled re-run of saved screen → diff vs prior → alert only on new matches.
- `billing`: Checkout → verified webhook → entitlement set/cleared → gates depth of results.
- `onboarding`: free lookup, see partial report, connect Telegram + pay to unlock.

**Agent:**
- Evaluates ONE candidate against the user's criteria; returns structured `ScreenResult` via **forced tool use** (see `/reference/agent_structured_output.py`).
- Inputs: structured candidate + `FinancialSnapshot` + criteria. Never raw filings.
- Guardrails: never invent figures; missing data → flag, don't guess; output must validate against schema or it is retried, never stored.

**Tools:** EDGAR discovery client, MCP financial reader, Stripe handlers, `telegram_send`, typed DB repositories.

---

## 4. Frozen contracts (full schemas in `/specs`)

**Models (`/specs/SCHEMA.md`):**
- `User`: id, email, telegram_chat_id, stripe_customer_id, entitlement_status
- `ScreenRequest`: id, user_id, keywords, sic_codes[], revenue_min/max, deal_size, custom_flags[], schedule(none|daily), status(queued|running|complete|failed)
- `Candidate`: id, screen_id, name, cik, ticker, source_url, status(found|enriched|screened|failed)
- `FinancialSnapshot`: revenue, revenue_prior, growth_rate, operating_income, ebitda_approx, gross_margin, total_debt, period_end, fiscal_year, verified=false, source_url
- `ScreenResult`: id, candidate_id, score(0–100), rationale, flags[], financial_snapshot, recommended(bool)

**API (`/specs/API.md`):**
- `POST /screens` → create + enqueue (entitlement checked: free tier = 1 lookup)
- `GET /screens/{id}` → status + progress counts
- `GET /screens/{id}/results` → **partial results while running** (supports progressive fill); free tier gets top 3 + redacted financials, gating applied server-side
- `POST /alerts/config`, `POST /billing/checkout`, `POST /billing/portal`
- `POST /webhooks/stripe` → raw-body signature verification + idempotency (see `/reference/stripe_webhook.py`)

**MCP tools (`/specs/MCP_TOOLS.md`):** `search_companies(query|sic)`, `get_company_facts(cik)`, `read_financial_sheet(cik)` → `FinancialSnapshot`

**Telegram (`/specs/ALERTS.md`):** `{ screen_id, summary, top_matches: [{name, score, source_url}] }`

---

## 5. Front-end UX spec — the BuiltWith model

The reference experience is builtwith.com: **one input, instant-feeling rich report, shareable permalink, freemium gate.** Replicate the pattern, not the styling.

**`/` (home):** Hero with a single large search input ("Enter an industry, niche, or keyword…") + a Screen button. Below it, 4–5 example chips ("solar energy", "pet care", "logistics software") that fill the input on tap. Nothing else above the fold. No login required for the first lookup.

**`/screen/{id}` (the report — this page IS the product):**
- Header: query, run date, status pill, copyable share URL.
- Summary band: candidates found, screened count (live), top score, avg score.
- Ranked candidate rows: score badge (color-coded), company name + ticker, one-line rationale, mini financial snapshot (revenue, growth, margin — each with source link), flag chips. Row expands to full rationale + full snapshot.
- **Progressive fill:** rows appear as skeletons when candidates are found and populate as each screen completes. Implemented by polling `GET /screens/{id}/results` every 2s until terminal status (hook in `/reference/useScreenResults.tsx`). **No websockets/SSE in v1** — polling is fewer failure modes for identical perceived UX.
- **The gate:** free users see top 3 rows fully; remaining rows render as locked placeholders (count + blurred bar + "Unlock full report"). The data for locked rows is NEVER in the API response.
- Why this works: BuiltWith feels instant because its data is pre-indexed. Ours can't be — progressive fill converts a 60–90s pipeline into something that feels alive in <3s.

**Other routes:** `/screens` (history), `/settings` (Telegram connect, billing portal link). That's all of v1.

---

## 6. Reference implementations — COPY, DO NOT REWRITE

The files in `/reference` solve the problems a code model reliably gets wrong. In any session touching these areas: read the reference file, copy it into the module, adapt imports only. Do not "improve" the logic.

- `edgar_client.py` — SEC headers + global rate limiter + ticker universe + **full-text search discovery** (keyword → CIKs). Models invent wrong endpoints and omit the mandatory User-Agent, then get IP-blocked.
- `xbrl_normalize.py` — **the hardest code in the repo.** Turns `companyfacts` XBRL into a clean `FinancialSnapshot`: GAAP tag fallback chains, fiscal-year duration filtering, dedupe-by-latest-filing. Naive code silently returns quarterly values as annual revenue or picks a deprecated tag — garbage that *looks* right.
- `stripe_webhook.py` — raw-body signature verification (FastAPI must NOT parse JSON first) + event idempotency table. The #1 most-botched integration pattern.
- `agent_structured_output.py` — forced tool-use for guaranteed-structure LLM output + Pydantic validation + bounded retry. Without this, "parse the JSON from the reply" breaks in production weekly.
- `concurrent_screening.py` — bounded-concurrency screening that persists each result as it lands (powers progressive fill) and isolates per-candidate failures.
- `gating.py` — server-side freemium truncation/redaction.
- `useScreenResults.tsx` — React polling hook with terminal-state stop + locked-row rendering.

---

## 7. Repo structure

```
/
  CLAUDE.md  README.md  ARCHITECTURE.md  .env.example  .gitignore
  /docs       WAT.md  DECISIONS.md
  /specs      SCHEMA.md  API.md  MCP_TOOLS.md  ALERTS.md   # FROZEN
  /reference  (see §6 — copy-source, never imported directly)
  /temp       /outputs  /resources   # staging only; gitignored; .gitkeep in each
  /backend    CLAUDE.md (routing table)  /app(api, agent, sources, billing, alerts, db, jobs)  /tests
  /mcp-server CLAUDE.md (routing table)  /tests
  /frontend   CLAUDE.md (routing table)  (Next.js + Tailwind + shadcn/ui)
```

`.gitignore` minimum: `.env`, `/temp/`, `__pycache__/`, `*.pyc`, `.DS_Store`, `node_modules/`, `.next/`.
Module CLAUDE.md files are auto-loaded by Claude Code when working in that directory — the routing tables enforce themselves.

---

## 8. Build sequence — ISOLATED SESSIONS (anti-context-rot plan)

One step = one fresh Claude Code session. Each session reads ONLY: this file + relevant `/specs` + relevant `/reference` + that module's CLAUDE.md. Build, test, commit, **clear**.

0. **Skeleton + freeze.** Repo structure, all `/specs`, module CLAUDE.md files (split from `MODULE_CONTEXTS.md` — they contain the routing tables), `/reference` files (split from `REFERENCE_IMPL.md`), `.env.example`, `.gitignore`, `/temp` with `.gitkeep`s. **Scaffold must be idempotent and non-destructive:** `mkdir -p` for dirs; for files, write only if absent (`[ ! -f path ] && cat > path`); NEVER overwrite an existing CLAUDE.md, .env, or any non-empty file. Commit.
1. **MCP server.** Copy `edgar_client.py` + `xbrl_normalize.py`; expose the three MCP tools. Test: `read_financial_sheet` on real CIKs (e.g. a known 10-K filer) returns a valid `FinancialSnapshot` with believable annual revenue.
2. **Backend pipeline.** Models + `run_screen` with mocked source/financials, then wire MCP. Copy `concurrent_screening.py`. Verify partial results appear row-by-row in the DB mid-run.
3. **Screening agent.** Copy `agent_structured_output.py`; write rubric + system prompt. Test: malformed model output never reaches the DB; missing financials produce flags, not invented numbers.
4. **Stripe.** Copy `stripe_webhook.py`. Hosted Checkout + Portal + entitlement. Copy `gating.py` into the results endpoint. Test with Stripe CLI webhook forwarding, including a replayed event (must no-op).
5. **Telegram.** Against `ALERTS.md`.
6. **Frontend.** Against `API.md` with mocked backend. Build §5 exactly: hero, report page, progressive fill (copy `useScreenResults.tsx`), server-driven locked rows.
7. **Integration.** Wire everything; run the full happy path: keyword → live report filling in → paywall → unlock → Telegram alert.
8. **Hardening ("try to break it").** Rate-limit storms, empty/weird filings (foreign private issuers, missing tags), webhook replay, expired entitlement mid-screen, EDGAR downtime. Then efficiency, output-quality/brand, and performance passes (framework steps 6–10).

---

## 9. Context + token discipline (every session)

- Repo is the memory; reference files by path, never re-paste them into chat.
- Deterministic parsing before any LLM call; the agent sees only `FinancialSnapshot`-level data (~300 tokens/candidate, not 300KB of XBRL).
- Plan mode / sub-agents for exploration; only summaries return to build context.
- Prompt caching on the agent's stable system context (rubric + criteria) — the screening loop re-pays only per-candidate tokens.
- Checkpoint + clear between §8 steps. Keep this file pruned.

---

## 10. Claude Code: strengths vs. risks here

**Lean on it for:** scaffolding, FastAPI glue, MCP wiring, the Next.js UI from §5, Telegram, docs, tests.
**It will fail without `/reference` on:** XBRL normalization, EDGAR etiquette, Stripe webhook handling, structured LLM output. That's why the reference code exists — copy it.
**You own:** the screening rubric (domain judgment), live verification against today's EDGAR/Stripe behavior, and prod reliability. Code that looks right ≠ code proven right.

---
> Source: [austinkyle/public-companies-data-scraper](https://github.com/austinkyle/public-companies-data-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
