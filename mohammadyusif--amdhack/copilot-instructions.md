## amdhack

> AI-powered alternative credit scoring for Saudi freelancers, embedded inside

# Mihan (مِهَن) — Claude Code Project Context

AI-powered alternative credit scoring for Saudi freelancers, embedded inside
Alinma Bank's app. Built for the AMAD 2026 hackathon (Alinma × Tuwaiq, July 16–18, Riyadh).
Scores income capacity from Open Banking cash-flow data because freelancers have
empty SIMAH files and no Mudad salary record.

**Status: feature-complete.** Backend (FastAPI, port 9000) and frontend
(Next.js 16 App Router, port 3000) are both done and demo-ready.

## Running

- Docker: `docker compose up --build` (both services)
- Windows native: `.\start.ps1`
- Manual: `cd backend && python -m uvicorn main:app --reload --port 9000` + `cd frontend && npm run dev`
- Tests: `cd backend && python -m pytest tests/` (132 cases: scoring, factor derivation, PII exclusion, statement import, entity resolution, import explanation/roadmap, regulatory XAI, forward-outlook predictive model, underwriting agent incl. Arabic-chip parity — must stay green)
- CI: `.github/workflows/ci.yml` runs tests + API smoke + docker build on push

## Key facts

- `backend/.env` (gitignored) holds real Wathq API credentials. Without it,
  everything falls back to simulation — demo still works but the live-proof
  button shows `"live": false`. See README Quick Start for the format.
- **Wathq is LIVE** (`backend/wathq_api.py` → api.wathq.sa sandbox). Lean,
  SIMAH, and Nafath are simulated in the demo. **Lean is NOT license-gated**:
  Lean Technologies is the first SAMA-licensed Open Banking provider (Major
  Payment Institution licence, Mar 27 2026) — Open Banking, incl. the AIS rail
  Mihan uses, has graduated from SAMA's regulatory sandbox to a licensed
  activity, so access for this platform depends on a **commercial bank-agent
  agreement** rather than waiting on regulatory clearance. SIMAH and Nafath
  remain genuinely license-gated. All use try-real-then-fallback.
- The Trial-tier Wathq sandbox returns one fixed record with the company name
  masked with literal `x` chars for ANY CR — that's why the persona narrative
  uses simulated names while `/wathq-live-proof` shows the raw live response.
- Determinism matters on stage: simulated data seeds use `zlib.crc32`, never
  the built-in `hash()` (randomized per process → scores would change across restarts).
- Frontend API base is `NEXT_PUBLIC_API_URL` (default `http://localhost:9000`),
  defined once in `frontend/lib/config.ts`.
- **Factors are derived, not hardcoded**: `income_stability` (CV over zero-filled
  18-month window) and `client_diversity` (HHI) are computed live from transactions
  in `backend/factor_analysis.py` on every scoring call. Persona composites:
  mohammad 81.2 GREEN / noura 57.9 YELLOW / fahad 36.5 BUILDING.
- **AI payload is zero-PII by construction**: `backend/ai_privacy.py` is the single
  source of truth for what reaches Claude (five scores + tier, nothing else) —
  used by both `generate_cache.py` and `/ai-privacy-proof`. Never add profile
  fields to `build_ai_prompt`; `tests/test_ai_privacy.py` fails on any leak.
- **Real-statement importer** (the "your cash flow is simulated" answer):
  `backend/statement_pdf.py` (offline CLI) parses a real Saudi retail bank-statement
  PDF and anonymizes it AT ingestion — names/accounts/cards/refs stripped, senders
  pseudonymized, fail-closed `assert_no_pii` scan. The importer is a **medallion
  pipeline**: bronze (raw parse, in-memory only) → silver (**entity resolution** —
  narration variants and multi-rail payments of the same counterparty collapse to
  ONE `ENTITY-` pseudonym) → gold (scoring). Deterministic rules only — Claude
  never touches ingestion. Entity keying is deliberately split by risk direction:
  `entity_key()` uses the FULL significant-token set (order-independent, legal
  suffixes like CO/LLC/EST dropped, descriptors like TRADING/HOLDING kept because
  they distinguish) so different clients sharing a first name never over-merge;
  `is_self()` uses a lossy consonant skeleton (MHMD, AL/EL prefix dropped) so
  spelling/transliteration variants of the ACCOUNT HOLDER are still caught and
  excluded — needing ≥2 token matches so a shared first name never triggers it.
  Arabic narrations are NFKC-folded from presentation forms so Arabic sender names
  tokenize instead of vanishing. All self-transfer variants collapse to the single
  holder in `silver_meta`. See `tests/test_statement_import.py::TestEntityResolution`.
  `POST /import-statement` scores
  the anonymized JSON through the same VANC pipeline with **4 of 5 factors computed
  live** (stability CV, diversity HHI, expense ratio, savings) plus an integrity
  check against the statement's own printed totals. The consented anonymized
  statement for the demo lives in gitignored `team/real_statement_anonymized.json`
  (bring it to the venue like the `.env`); it scores BUILDING/no-loan — honest, and
  it demos the self-transfer / cash-deposit income-exclusion controls. Never commit
  raw statements or statement PDFs. `POST /import-statement-pdf` is the
  direct-upload variant: it accepts the raw PDF, runs the SAME anonymizer
  in memory only (never persisted, fail-closed PII scan still gates scoring),
  and returns the result plus the anonymized statement (zero-PII, reused by
  the UI for live-AI regeneration). Needs `pdfplumber` + `python-multipart`.
  The frontend screen is `/import` («جرّب ملفك»,
  entry card on `/demo`) and accepts BOTH the raw PDF and the pre-anonymized
  JSON; the response includes an evidence-grounded roadmap
  (`improvement_roadmap.py` with `import_evidence`) and a bilingual explanation
  (`statement_explain.py`: template by default, `?live_ai=true` tries Claude with
  the same zero-PII payload — five scores + tier — and falls back on any failure).
- **Decision-intelligence layer** (three modules on top of the score, all
  deterministic, all reusing the score/factor evidence — no new PII surface):
  * `regulatory_xai.py` — auditor-ready justification: EXACT principal-factor
    decomposition (composite = Σ weight×score, so attributions are exact, not
    SHAP-on-a-black-box), a fair-lending adverse-action notice on decline/DBR
    compression, the SAMA Art. 14(b) DBR arithmetic, and an INPUT-LEVEL
    fairness attestation (no protected attribute enters the score or the AI
    payload). Deliberately NOT a statistical disparate-impact audit. Also emits
    a `cautionary` block (margin of transparency: WATCH_<factor> for 55<score≤65
    drift + MARGINAL_APPROVAL when composite clears the 55 line by <5) so
    approved-but-borderline files still get an explanation. Each record carries
    a deterministic SHA-256 `content_hash` (reproducible from the score);
    `point_in_time_stamp()` binds it to issued_at + record_hash at the endpoint
    and writes it to the append-only audit ledger (tamper-evident, NOT
    blockchain). Endpoint `GET /profiles/{id}/regulatory-explainability`; also
    embedded in `/import-statement`. Frontend: `RegulatoryXAIPanel`.
  * `predictive.py` — `forward_outlook()`: a 6-month forward default
    probability from a TRANSPARENT logistic with PUBLISHED, fixed coefficients
    over interpretable [0,1] risk signals (volatility, income-trend slope,
    concentration, savings, DBR utilisation, Wathq registry flag) fused with
    SIMAH status ("Hybrid Analysis"). Fully decomposed + reproducible; labelled
    as a decision-support early-warning signal, NOT a trained PD model. Tune
    coefficients/intercept in `predictive.py` only (persona targets: mohammad
    ~6% LOW / noura ~29% ELEVATED / fahad ~45% HIGH). Endpoint
    `GET /profiles/{id}/forward-outlook`; embedded in `/import-statement`.
    Frontend: `ForwardOutlookPanel`.
  * `underwriting_agent.py` — auto-drafts an underwriter recommendation and
    answers officer chat, grounded ONLY in a zero-PII aggregate built by
    `build_agent_context()`; `assert_context_clean()` fail-closes on any long
    digit run or raw-data key, and the opt-in live-Claude path gets the SAME
    aggregate. Endpoints `GET /profiles/{id}/underwriter-recommendation` and
    `POST /agent/ask` ({profile_id, question, live_ai}); draft embedded in
    `/import-statement`. Frontend: `UnderwriterAgent` (the one client
    component in `/banker/[id]`). Chat is MULTI-INTENT: `answer_question`
    composes ALL matching intents (up to 3) with union grounding, plus a
    `what_if` handler for curveballs (directional, no invented numbers).
    Chat is BILINGUAL: every template handler returns `answer_en` AND
    `answer_ar` (UI renders by language, falls back to EN for live answers),
    and keyword matching normalizes Arabic first (strip harakat/shadda/
    tatweel, fold alef variants) with singular AND broken-plural stems —
    "خطر/مخاطر/أخطار", "شرط/شروط" — because the UI's own Arabic suggestion
    chips otherwise fall through to the generic summary (this shipped broken
    once; `test_underwriting_agent.py` Arabic-parity tests pin it). The
    context includes a `volatility` block (μ, σ, μ−1.5σ income, and the
    conservatism haircut) so the agent can explain the VANC penalty — driven by
    `MihanScore.vanc_mean` / `vanc_sigma`.
- Public pitch materials live in `docs/pitch/`: deck outline, competitive
  landscape, unit economics, deck screenshots, and the backup videos
  (`mihan_demo_backup.webm` — persona flow; `mihan_import_demo.webm` —
  real-statement import incl. the medallion strip; `mihan_officer_demo.webm`
  — officer decision-intelligence layer incl. the Arabic agent chat).
- Backstage prep (demo script, day-of checklist, team cheat sheet, 72h plan)
  lives in the gitignored `team/` folder — kept out of the judged repo on
  purpose; it contains on-stage framing/Q&A coaching. Not pushed.

## Regulatory constraints (do not deviate)

- **DBR cap 45%** of total monthly income — SAMA Responsible Lending Art. 14(b).
  (33.33% applies only to employer payroll deduction — NOT here.)
- Repayment capacity = worst-month income basis (v1) or VANC μ−1.5σ (v2), never the average.
- Tiers: GREEN ≥75 (SAR 60K/24mo/8.2%), YELLOW ≥55 (SAR 25K/18mo/11.5%), BUILDING <55 (no loan).
- Weights: expense 30 / income-stability 25 / diversity 20 / savings 15 / contract-verification 10.
- Thin SIMAH + Mihan ≥75 → exception sandbox (auto-reject bypassed, routed to officer).
- The PDF is a "Cash Flow History Statement" — NEVER call it "Proof of Income"
  anywhere user-facing (legal liability distinction). Don't change the disclaimer in pdf_gen.py.
- Every scoring event goes to the SQLite audit log with endpoint + factor breakdown.
- No auto-approval — decisions always route through the officer dashboard.

## What NOT to do

- No pages/ directory — App Router only. Next.js 16 has breaking changes:
  read `frontend/node_modules/next/dist/docs/` before writing frontend code.
- No authentication — demo only.
- Don't hardcode Arabic text in JS — use the translations object (`frontend/lib/i18n.ts`).
- Don't show raw SIMAH score numbers (all personas return None).
- Personas (mohammad=GREEN, noura=YELLOW, fahad=BUILDING) are scripted — changing
  factor inputs or simulation data breaks the demo narrative and the tests.

---
> Source: [MohammadYusif/amdhack](https://github.com/MohammadYusif/amdhack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
