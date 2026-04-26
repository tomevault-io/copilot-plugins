## drug-knowledge-assistant

> RAG system for drug-related Q&A. 50 drugs, 100 docs, hybrid 3-layer retrieval.

# CLAUDE.md -- AI Agent Instructions for Drug Knowledge Assistant

## Project Overview
RAG system for drug-related Q&A. 50 drugs, 100 docs, hybrid 3-layer retrieval.
Live demo hosted on HuggingFace Spaces. GitHub Pages site in `docs/`.

## Code Conventions
- Python 3.10+, managed via uv
- Formatting: ruff (line-length 88, double quotes)
- Linting: ruff check (E, W, F, I, UP, B rules)
- Testing: pytest (tests/ directory)
- API keys go in environment variables, NEVER hardcoded
- All retrieval functions should return a DataFrame with a `score` column
- Fallback answers must work without LLM

## Dev Commands
```bash
uv sync                  # install dependencies
uv run pytest            # run tests
uv run ruff format .     # format code
uv run ruff check --fix  # lint + autofix
```

## Key Architecture
- Hybrid 3-layer retrieval (drug match + TF-IDF + keyword match)
- Drug-pair sibling linking (always pull both docs for a drug)
- Query classification before retrieval (6 types)
- Few-shot prompting with query-type-aware templates
- Unanswerable detection via score threshold (0.10)
- Structured fallback from drug_knowledge.csv (works without LLM)

## Deployment
- **Gradio app**: `app.py` (deployed to HF Spaces, not in GitHub deps)
- **HF Space**: https://huggingface.co/spaces/Fo-zh/drug-knowledge-assistant
- **API keys**: stored as HF Space secrets (`OPENROUTER_API_KEYS`), never in code
- **GitHub Pages**: `docs/index.html`

## Cross-Session Context
- `memory-bank/` contains Cursor memory bank files for AI continuity
- Read `memory-bank/activeContext.md` and `memory-bank/progress.md` for current state

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/H-furkan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
