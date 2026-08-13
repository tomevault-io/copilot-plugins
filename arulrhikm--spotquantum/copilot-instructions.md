## spotquantum

> SpotQubit is a quantum circuit intelligence platform ("the quantum algorithms

# SpotQubit — agent guide

## What this project is
SpotQubit is a quantum circuit intelligence platform ("the quantum algorithms
stack"): circuit/paper in → MRI analysis → fault-tolerance & resource
estimation → cross-vendor optimization/comparison → advantage screening.
Single Next.js app (marketing landing + `/workspace/`) served by a FastAPI
backend at one URL, or split Vercel (frontend) + Render (backend).
Deeper subsystem notes live in `docs/brain/` (hub: `SpotQubit Home.md`).

## Working efficiently (context hygiene)
- **Delegate broad exploration to subagents** (built-in Explore, project agents
  in `.claude/agents/`, global `researcher`) so large reads stay out of the main
  context; you get back just the summary.
- **Query first, scan later**: use the graphify code graph and `docs/brain/`
  before grepping/reading files wholesale.
- Specialist agents declare `model:` frontmatter — keep grunt/visual work on
  cheaper tiers.

## Repo layout (current)
```
SpotQuantum/
├── backend/
│   ├── main.py                  # FastAPI entry: middleware, caches, v1 routes
│   ├── api/v2.py                # /api/v2 analysis+designer routes (paper cache lives here)
│   ├── api/chatbot.py           # assistant endpoint (+ chatbot_kb.py knowledge base)
│   ├── guardrails.py            # per-IP rate buckets + daily LLM cap (visible in /health)
│   ├── stats.py                 # usage counters (/api/v2/stats) + waitlist capture
│   ├── analysis/                # metrics, hardness, fault_tolerance, devices,
│   │                            #   advantage (+ registry), scaling, feasibility,
│   │                            #   cost_model, diskcache, pipeline
│   ├── paper_parser/            # arxiv fetch, PDF text, LLM extraction, circuit gen
│   ├── agents/                  # LLM provider chain (Gemini primary, keyless fallback)
│   └── demo_circuits/           # ql_*.qasm demo circuits
├── frontend/quantumlens/        # Next.js app (landing + workspace); static export to out/
├── scripts/                     # launch.py, check.py + audit harnesses (see scripts/README.md)
├── tests/                       # pytest suite (markers: slow, smoke)
├── docs/                        # DEPLOY, brain/, pitch/, correctness_audit/
│   └── doc-map.json             # code→docs sync map (see Doc sync below)
└── pyproject.toml
```
The legacy vanilla-JS UI, agent stack, and PaperLens deploy were deleted in the
July 2026 launch sprint (`v0-pre-launch` tag predates it).

## Rules that still matter
1. **Honesty first.** Never show fabricated numbers: papers with no derivable
   circuit return `metrics: null` + a grounded reason; estimates carry
   provenance labels (`data_source`, snapshot badges); glassiness labels are
   literature-calibrated, never claimed as computed.
2. New v2 API routes go in `backend/api/v2.py`; Pydantic models in
   `backend/analysis/models.py` — never inline them in handlers.
3. Analysis functions stay pure (no I/O); I/O lives in route handlers and
   `paper_parser/`.
4. LLM usage is free-tier only (`GOOGLE_API_KEY`, Gemini), with a keyless
   deterministic fallback — everything must degrade gracefully without a key.
5. No em dashes in user-visible strings.
6. Key data contracts (field names are frozen): `CircuitIR`, `AnalysisReport`
   (metrics is nullable), `PaperExtraction`, `HardnessClass` enum
   EASY | MEDIUM | HARD | INTRACTABLE.

## Doc sync (keep docs honest)
`docs/doc-map.json` maps code globs → the docs that describe them. The
project-root hook `../.claude/hooks/doc_sync.py` prints "possibly stale" doc
advisories at session start and turn end (advisory only, never blocks; also
step 4 of `/verify`). When you change a subsystem, update the mapped doc(s) in
the same change — the brain notes especially (`/update-brain` automates the
candidate list).

## How to run
```bash
python scripts/launch.py              # build static export + serve app+API at :8000
python scripts/launch.py --full       # dev mode: backend :8000 + Next dev :3000
python scripts/launch.py --api-only   # API only
```

## Verification
```bash
python -m pytest tests/ -m "not slow" -q      # CI gate parity (~30s)
cd frontend/quantumlens && npx tsc --noEmit && npm run build

# Live-server harnesses (honor BASE / SPOTQUBIT_URL = http://127.0.0.1:<port>):
python scripts/integration_api.py     # transpile + analyze smoke (CI gate)
python scripts/industry_personas.py   # 15 enterprise teams (always exits 0 — read stdout)
python scripts/paper_audit.py         # offline paper feasibility fixtures (exits 1)
```
Exact-value ground truths (metrics, FT closed form, device model, Qiskit parser
oracle) live in `tests/test_correctness.py` and are documented in
`docs/correctness_audit/AUDIT.md`.

## Environment variables
`GOOGLE_API_KEY` (Gemini free tier; optional), `ALLOWED_ORIGINS`,
`DAILY_LLM_CAP`, `RATE_LIMIT_PER_MIN`, `SPOTQUBIT_SKIP_WARMUP`,
`SPOTQUBIT_CACHE_DIR`. See `env.example` for the full list and `docs/DEPLOY.md`
for the deploy shape (Render Docker + Vercel static export).

---
> Source: [arulrhikm/SpotQuantum](https://github.com/arulrhikm/SpotQuantum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
