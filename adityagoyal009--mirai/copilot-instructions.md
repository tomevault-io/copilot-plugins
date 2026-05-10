## mirai

> Always read DESIGN.md before making any visual or UI decisions.

## Design System
Always read DESIGN.md before making any visual or UI decisions.
All font choices, colors, spacing, and aesthetic direction are defined there.
Do not deviate without explicit user approval.
In QA mode, flag any code that doesn't match DESIGN.md.

Key rules:
- HTML reports use Instrument Serif (display), DM Sans (body/data), Source Serif 4 italic (agent quotes)
- Primary navy: #0f2440. Accent blue: #2563eb. HIT green: #059669. MISS red: #dc2626.
- Agent quote cards use tinted backgrounds (green for HIT, red for MISS) with serif italic text
- Score bars use red-amber-green gradients, not solid fills
- Dashboard (pixel art war room) has its own separate design system — do not apply report design rules to it

## Pipeline Architecture
- Research: Claude Code CLI primary (6-phase WebSearch/WebFetch), OpenClaw fallback, Gemini grounded search final fallback, no degraded BI web-less fallback in the live pipeline
- Council: 11 models across 8 families. Karpathy 3-stage pattern (individual → peer review → chairman)
- Chairman: Opus primary, Qwen3.5 397B (NVIDIA) fallback
- Swarm: 50 agents across 6 NVIDIA NIM models, 8 concurrent workers (40 RPM safe)
- Swarm personas: stage-aware + target-market-aware, with wildcard caps for B2B / industrial runs
- OASIS: 4-round market simulation, auto-enabled, uses swarm agents as panelists, and tracks trajectory from the pre-simulation baseline
- All data flows WITHOUT truncation — full research context to council and swarm
- Hallucination guard runs on every swarm agent reasoning
- Bias controls: neutral GEO_BEHAVIORAL, no MBTI scoring directives, DELIBERATION_WEIGHT=1.0, industry weights capped 1.5x, stage vocabulary normalized, verdict override removed (advisory note only)
- Final verdict: shared REST/WebSocket numeric blend of council + swarm + OASIS
- Report: HTML via `generate_html_report()`. Opens in new tab. No Playwright/Chromium dependency. Fact-check data may come from top-level, prediction, or swarm payloads.
- Async API: `/api/bi/analyze` returns job_id, poll `/api/bi/job/{id}` every 15s, internal-auth protected

## Website Form
- 122 industries (searchable), 789 keywords (multiselect, max 15), 195 countries (searchable)
- Rating-critical structured fields now include buyer context (`endUser`, `economicBuyer`, `switchingTrigger`), proof counts (`loiCount`, `pilotCount`, `activeCustomerCount`, `paidCustomerCount`, `monthlyRevenueValue`, `growthRate`), pricing/GTM (`pricingModel`, `startingPrice`, `salesMotion`, `typicalContractSize`), implementation (`implementationComplexity`, `timeToValue`), substitute / evidence / founder-fit / risk fields
- Backend validation: enum checks, URL format, 100K char limit, atomic Prisma transactions
- Structured fields passthrough to backend (skips LLM extraction)
- The structured intake must flow into research, swarm persona selection, OASIS context, and report transparency. Do not treat these as cosmetic extra fields.
- Retry queue: 3 retries with 10s delay, health check before each attempt, 60 min timeout
- Restart recovery rebuilds safe `queued` and `reviewing` jobs into the queue
- Founder APIs expose `status_message` instead of raw `admin_notes`
- Internal website → swarm auth uses a dedicated `MIRAI_INTERNAL_API_KEY`

## Model Routing (llm_client.py)
- `@cf/` models → Cloudflare Workers AI (kept but not in active council)
- Groq models → `_call_groq()` direct REST API
- Cerebras models → `_call_cerebras()` direct REST API
- SambaNova models → `_call_sambanova()` direct REST API
- Mistral models → `_call_mistral()` direct REST API
- NVIDIA NIM models → `_call_nvidia()` direct REST API
- Claude models → `_call_claude_cli()` subprocess (7 min timeout), gateway fallback
- OpenAI/GPT models → `_call_codex_cli()` subprocess, gateway fallback
- OpenClaw models → `_call_openclaw_gateway()` port 18789

## Key Decisions
- No Gemini in council or swarm (unreliable CLI, hangs). Gemini used for research fallback only.
- No backtest with known companies (temporal mismatch)
- No research degraded mode — if Claude CLI, OpenClaw, and Gemini all fail, the pipeline stops
- Swarm agents are blind to council scores (independent evaluation)
- Frontend sends structured fields directly (bypasses LLM extraction for form data)
- Pipeline bias fixes: no geographic stereotypes, no personality-based scoring, equal deliberation weight, no verdict hard-overrides, capped industry dimension weights, softened data quality penalty
- REST API matches dashboard pipeline 1:1 (structured passthrough, blind scoring, Claude/OpenClaw/Gemini research chain, council deep, swarm 50, OASIS, HTML report)
- Council fact-checking should operate on council reasoning text, not raw research JSON
- `avg_scores["overall"]` is part of the swarm contract and is used by final verdict blending

---
> Source: [adityagoyal009/Mirai](https://github.com/adityagoyal009/Mirai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
