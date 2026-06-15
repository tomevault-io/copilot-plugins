## resume-pipeline-v2

> A multi-agent pipeline that compares a resume against a job description, identifies keyword gaps, suggests targeted edits, and scores the output quality — with full auditability and human-in-the-loop review for low-confidence cases.

# Resume Keyword Alignment Pipeline

A multi-agent pipeline that compares a resume against a job description, identifies keyword gaps, suggests targeted edits, and scores the output quality — with full auditability and human-in-the-loop review for low-confidence cases.

## Architecture

The pipeline follows a **Gatekeeper → Worker → Judge** pattern:

```
PDF → Preprocessor → Gatekeeper → Worker → Schema Validation → Judge → DB
                         ↓                                          ↓
                    (fast-pass)                            (human review queue)
```

| Stage | Role | LLM? |
|-------|------|-------|
| Preprocessor | Extract text from PDF resume | No |
| Gatekeeper | Lemmatize & match keywords; pass/fail gate | No (deterministic, spaCy) |
| Worker | Identify keyword gaps, suggest resume edits | Yes (Claude Sonnet) |
| Schema Validation | Validate Worker output shape (Pydantic) | No |
| Judge | Score Worker output quality, flag for human review | Yes (Claude Sonnet) |

### Key design principles

- **Determinism where possible.** The Gatekeeper uses spaCy lemmatization and exact/stem matching — no LLM involved. Same inputs always produce the same gate result.
- **Full auditability.** Every pipeline run is written to SQLite, including all inputs, outputs, token usage, cost, and errors.
- **Human-in-the-loop.** Low-confidence Judge scores flag the run for human review via the review API. Nothing is silently discarded.
- **No hardcoded prompts.** All LLM prompts live in `/prompts/` as `.txt` files and are loaded at runtime.
- **No hardcoded model names.** Model identifiers come from environment variables only.
- **Schema as contract.** Pydantic models sit between Worker output and Judge input — malformed Worker output is caught before the Judge ever runs.

## Project Structure

```
resume-pipeline-v2/
├── prompts/
│   ├── worker_system.txt   # Worker system prompt
│   ├── worker_user.txt     # Worker user prompt (templated with resume + JD + gaps)
│   ├── judge_system.txt    # Judge system prompt
│   └── judge_user.txt      # Judge user prompt (templated with Worker output)
├── sample_data/
│   ├── sample_resume.txt
│   └── sample_jd.txt
├── src/
│   ├── config.py         # Settings loaded from env vars (model names, thresholds, DB path)
│   ├── database.py       # SQLite schema + all DB read/write helpers
│   ├── validator.py      # Pydantic models for Worker output and Judge scores
│   ├── gatekeeper.py     # Deterministic keyword matching (spaCy lemmatization)
│   ├── worker.py         # LLM call: gap analysis + edit suggestions
│   ├── judge.py          # LLM call: quality scoring + human-review flagging
│   ├── pipeline.py       # Orchestrates the full Gatekeeper → Worker → Judge flow
│   ├── preprocessor.py   # PDF text extraction (pdfplumber)
│   └── review_api.py     # FastAPI endpoints for the human review queue
├── main.py               # CLI entry point
├── CLAUDE.md
└── requirements.txt
```

## Build Order

When scaffolding or rebuilding from scratch, implement in this order to respect dependencies:

1. `scaffold` — directory layout, `requirements.txt`, `.env.example`
2. `database` — SQLite schema and helper functions
3. `config` — env-var-backed settings (no defaults for model names or secrets)
4. `validator` — Pydantic models (Worker output schema, Judge score schema)
5. `gatekeeper` — spaCy lemmatizer + keyword matching logic
6. `prompts` — all `.txt` prompt templates
7. `worker` — LLM call using Anthropic SDK, loads prompt from `/prompts/`
8. `judge` — LLM call, validates Worker output via Pydantic before scoring
9. `pipeline` — orchestration: wires stages together, writes runs to DB
10. `preprocessor` — PDF extraction (pdfplumber), feeds into Gatekeeper
11. `main` — CLI (argparse or Typer) that drives the pipeline
12. `review_api` — FastAPI app exposing the human review queue

## Tech Stack

| Concern | Library |
|---------|---------|
| LLM calls | `anthropic` SDK |
| NLP / lemmatization | `spacy` (en_core_web_sm or lg) |
| PDF extraction | `pdfplumber` |
| Schema validation | `pydantic` v2 |
| Database | `sqlite3` (stdlib) |
| Review API | `fastapi` + `uvicorn` |
| Structured logging | `structlog` |
| Config / env | `python-dotenv` |

## Environment Variables

All model names and tuneable parameters come from env vars. No fallback defaults for model names — fail loudly if unset.

```
WORKER_MODEL=claude-sonnet-4-6
JUDGE_MODEL=claude-sonnet-4-6
DATABASE_PATH=./pipeline.db
JUDGE_CONFIDENCE_THRESHOLD=0.75   # below this → flag for human review
LOG_LEVEL=INFO
```

## Database Schema (key tables)

- `runs` — one row per pipeline execution: resume path, job description, timestamp, pass/fail, total tokens, estimated cost, error text
- `gatekeeper_results` — keyword match details per run
- `worker_outputs` — raw Worker JSON response per run
- `judge_scores` — Judge score, confidence, flag status per run
- `review_queue` — runs flagged for human review, with resolution status

## Coding Conventions

- All prompts loaded from `/prompts/<stage>_system.txt` and `/prompts/<stage>_user.txt` at call time, not module import time.
- Model names read from `config.py` (which reads from env) — never a string literal in a call site.
- Token usage and cost appended to the `runs` row after each LLM call returns.
- Errors caught at the pipeline level; written to `runs.error` and re-raised as structured exceptions — never silently swallowed.
- `structlog` used for all logging; log entries include `run_id` for traceability.
- Pydantic validation failures (between Worker and Judge) are treated as pipeline errors, logged, and flagged for human review.
- Tests live in `/tests/`; Gatekeeper is unit-tested with known lemma pairs; Worker and Judge are tested with recorded fixtures, not live LLM calls.

## Running the Pipeline

```bash
# Single resume against a job description
python main.py --resume path/to/resume.pdf --job-description path/to/jd.txt

# Start the review API
uvicorn src.review_api:app --reload
```

## Human Review API (FastAPI)

`GET  /review/queue`          — list runs flagged for review  
`GET  /review/{run_id}`       — full run detail (inputs + Worker output + Judge score)  
`POST /review/{run_id}/resolve` — mark reviewed with human decision and notes

---
> Source: [jacktyn/resume-pipeline-v2](https://github.com/jacktyn/resume-pipeline-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
