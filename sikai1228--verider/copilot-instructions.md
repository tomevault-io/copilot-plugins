## verider

> Read SPEC.md fully before writing code. It is the working spec; this file adds the pinned engineering contracts that keep components compatible.

# Verider — repo guide for Claude Code

Read SPEC.md fully before writing code. It is the working spec; this file adds the pinned engineering contracts that keep components compatible.

## Layout

- `db/migrations/` — append-only plain SQL, numbered `NNNN_name.sql`. Never edit an applied migration.
- `db/migrate.py` — minimal runner (tracks applied files in `schema_migrations`).
- `pipeline/` — Python package `verider_pipeline` (ingestion, screening, case generation, exports, refresh).
- `services/api/` — in-country review API (FastAPI, RBAC, audit).
- `services/score/` — `verider-score` cloud inference service (the only component that talks to model providers).
- `ui/` — React + TypeScript review UI.
- `eval/` — T1 eval harness (cloud side, never deployed in-country).
- `docs/` — architecture notes, runbooks.

## Pinned contracts (all components must agree)

### Database
- Dev DSN: `postgresql://verider:verider@localhost:5432/verider` (env `DATABASE_URL`).
- PostGIS required; SRID 4326 for all geometries.
- Enum values, exactly these strings:
  - `cases.tier_status`: `t1_pending | t1_cleared | t1_flagged | t2_pending | t2_done | t3_pending | t3_done`
  - `cases.model_verdict`: `discrepancy_real | match_found | uncertain`
  - `cases.official_decision`: `pending | approved | rejected`
  - `roll_entries.status`: `raw | normalized | matched | unmatched | retired`

### Audit log hash chain
- Row hash = `sha256(prev_hash || canonical_json(payload))` where `canonical_json` is JSON with sorted keys, no whitespace, UTF-8.
- Genesis `prev_hash` = 64 zero hex chars.
- The chain is computed by a DB trigger on insert; clients supply payload fields only. No UPDATE or DELETE grants on `audit_log`.
- Every API view or action writes one row with `actor, role, action, object_type, object_id, access_reason, request_id`.

### Score service API (in-country client -> verider-cloud)
- `POST /v1/score/batch`, bearer token (`SCORE_API_TOKEN`).
- Request: `{"run_id": str, "cases": [{"case_id": str, "features": {...}}]}`
- Response: `{"results": [{"case_id": str, "verdict": "discrepancy_real|match_found|uncertain", "confidence": float, "reasoning_summary": str, "cached": bool}], "cost_usd": float, "budget_state": {"run_spent_usd": float, "month_spent_usd": float, "halted": bool}}`
- Budget exhausted: HTTP 402, body `{"error": "budget_exhausted", "budget_state": {...}}`. Clients halt the queue and persist the halt state; never retry-loop past a 402.
- Feature payloads carry the minimum needed to score: geometry stats, neighborhood context, match candidates with scores. Never names, TINs, or raw roll rows.

### Config (env, see `.env.example`)
`DATABASE_URL`, `PILOT_WOREDA_LIMIT` (default 15), `SCORE_API_URL`, `SCORE_API_TOKEN`, `SCORE_RUN_BUDGET_USD`, `SCORE_MONTH_BUDGET_USD`, `ANTHROPIC_API_KEY` (score service only, never in-country).

## Working agreements

- Python 3.11+, pydantic v2, psycopg 3. Tests must pass without a live database (mark DB-integration tests to skip when `DATABASE_URL` is unreachable).
- Every pipeline stage: idempotent, resumable, `--dry-run`, structured JSON logs with a run id.
- Any path that can call a model API consults the budget service first and fails closed.
- Docker is not installed on this dev machine; keep compose files correct but verify by unit tests.

## Writing rules (all English text: UI copy, docs, errors, commit messages)

- Sentence case everywhere, including buttons, labels, and headings. No trailing periods on headings, buttons, or labels.
- No em dashes, no exclamation points, no ALL CAPS (including CSS `text-transform: uppercase`).
- Contractions in prose; verb-first, concrete, no filler ("in order to", "is capable of", "feel free to").
- "woreda", never "kebele". Client-facing strings never name data sources or say "satellite"; drone work is "drone imaging, at the government's option".
- Amharic is first-class; transliteration is for matching only. Display rule (founder decision, Aug 3, 2026): the review console renders ONE language at a time, English by default, with a footer language toggle; a bilingual data field shows the selected locale's script and falls back to whichever script the record carries. Originals are never discarded and taxpayer names always render in their original script. Paper deliverables (field sheets) keep both scripts. Clarification (orchestrator ruling, Aug 2, 2026, flagged for founder review): opaque identifiers such as case keys may contain a transliterated fragment, because a paper reference must be enterable on any keyboard; this never substitutes for display.

---
> Source: [sikai1228/verider](https://github.com/sikai1228/verider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
