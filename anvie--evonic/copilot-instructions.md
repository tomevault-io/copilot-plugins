## evonic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Evonic LLM Evaluator — a Flask web app that evaluates LLMs across multiple domains (conversation, math, SQL, tool calling, reasoning, health, needle-in-haystack, etc.) with 5 complexity levels each. It targets OpenAI-compatible APIs (llama.cpp, Ollama, vLLM, OpenRouter). The evaluation UI and many test prompts use Indonesian language.

## Commands

```bash
# Run the web server (default: http://localhost:8080)
python3 app.py

# Run headless evaluation from CLI
python3 run_headless.py --endpoint http://localhost:8080/v1 --model default

# Run all unit tests
pytest unit_tests/

# Run a single test file
pytest unit_tests/test_tool_call_evaluation.py -v

# Install dependencies
pip install -r requirements.txt
```

## Architecture

### Evaluation Flow

1. `app.py` receives `POST /api/start` → spawns background thread via `EvaluationEngine`
2. Engine iterates domains/levels, loads test definitions from `test_definitions/<domain>/level_<n>/test_*.json`
3. Builds system prompt using 3-layer hierarchy: domain → level → test (each layer supports "overwrite" or "append" mode)
4. **PASS 1**: Calls LLM for full response with reasoning
5. Routes to domain-specific evaluator strategy (Strategy Pattern)
6. **PASS 2** (if applicable): Calls LLM again to extract a clean answer in strict format
7. Evaluator scores the response → result saved to SQLite + JSON log
8. Frontend polls `/api/status` and `/api/test_matrix` for live updates

### Key Modules

- **`evaluator/engine.py`** — Singleton `EvaluationEngine` managing lifecycle, background threads, logging queue
- **`evaluator/scoring.py`** — `ScoringEngine` with weighted scoring (level N = weight N)
- **`evaluator/llm_client.py`** — OpenAI-compatible API client; handles thinking tag stripping (standard `<think>` and Gemma format)
- **`evaluator/answer_extractor.py`** — PASS 2 extraction with domain-specific templates and regex fallbacks
- **`evaluator/domain_evaluators.py`** — Factory/registry mapping domains to evaluator strategies
- **`evaluator/test_loader.py`** / **`test_manager.py`** — Load test definitions from filesystem; CRUD with file+DB sync
- **`evaluator/custom_evaluator.py`** — Custom evaluators in three modes: regex-only, LLM-prompt-only, hybrid
- **`models/db.py`** — SQLite persistence (evaluation_runs, test_results, individual_test_results)
- **`config.py`** — Central config loaded from `.env`

### Evaluator Strategies (`evaluator/strategies/`)

All inherit from `BaseEvaluator` (ABC) and return `EvaluationResult(score, status, details, extracted_answer, pass2_used)`:

| Strategy | Domains | PASS 2? | Key behavior |
|---|---|---|---|
| `KeywordEvaluator` | conversation | No | Keyword/regex matching with Indonesian fluency scoring |
| `TwoPassEvaluator` | math, reasoning, health | Yes | Extract clean answer via LLM, then validate against expected |
| `SQLExecutorEvaluator` | sql | Yes | Extract SQL, execute on SQLite, validate results (columns, rows, data quality) |
| `ToolCallEvaluator` | tool_calling | Yes | Validate tool calls against expected tools; supports chained multi-step calls |

### Test Definitions

```
test_definitions/
├── <domain>/
│   ├── domain.json          # Domain metadata, default evaluator, system_prompt
│   ├── level_<n>/
│   │   └── test_*.json      # Individual test with prompt, expected, evaluator_id
├── evaluators/              # Evaluator config JSONs (two_pass, keyword, regex_*, etc.)
└── tools/                   # Tool registry JSONs (OpenAI function schema + mock responses)
```

Evaluator configs in `test_definitions/evaluators/*.json` define behavior for: `two_pass`, `keyword`, `sql_executor`, `tool_call`, `natural_text_compare`, `hybrid_quality_rater`, and various `regex_*` patterns.

### Frontend

Vanilla HTML/CSS/JS with Jinja2 templates — no build step. Templates in `templates/`, static assets in `static/`.

## Coding Guidelines

### Frontend

- Always use **jQuery** for DOM manipulation, event handling, and AJAX calls — do not use vanilla `fetch` or `addEventListener` when jQuery equivalents exist.
- Always use **Tailwind CSS** for styling — do not write custom CSS unless absolutely necessary (e.g., animations not supported by Tailwind).
- No build step: jQuery and Tailwind are already bundled in `static/` — reference them from there, do not use external CDN links.

### Backend

- Minimize third-party dependencies — prefer Python stdlib over adding new packages.
- Before adding a new `pip` dependency, check if the same can be achieved with stdlib (e.g., use `sqlite3` over an ORM, `re` over a regex library, `json` over a serialization library).
- New packages must be justified and added to `requirements.txt`.

## Configuration

All config via `.env` (see `.env.example`). Key settings: `LLM_BASE_URL`, `LLM_API_KEY`, `LLM_MODEL`, `LLM_TIMEOUT`, `TWO_PASS_ENABLED`, `TWO_PASS_TEMPERATURE`. Per-domain evaluator overrides via `EVALUATOR_<DOMAIN>` env vars.

---
> Source: [anvie/evonic](https://github.com/anvie/evonic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
