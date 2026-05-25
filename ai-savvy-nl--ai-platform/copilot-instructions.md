## ai-platform

> Production-grade RAG + agents monorepo. Stack: Python 3.11, FastAPI, Qdrant, sentence-transformers, OpenAI + Anthropic.

# AI Platform — Claude Code context

## Wat dit is
Production-grade RAG + agents monorepo. Stack: Python 3.11, FastAPI, Qdrant, sentence-transformers, OpenAI + Anthropic.

## Structuur
- `apps/` — services (api, ingestion, agents, eval). Geen gedeelde state tussen apps.
- `libs/` — shared logic. Geen afhankelijkheden op `apps/`.
- `tests/unit/` — geen I/O, altijd snel. `tests/integration/` — mock vector store. `tests/eval/` — regression gates.

## Conventions
- Alle settings via `libs/utils/settings.py` (pydantic-settings). Nooit `os.getenv` direct.
- Logging via `structlog`. Nooit `print()` in library code.
- LLM calls altijd via `libs/llm/` — niet direct `openai` of `anthropic` importeren in `apps/`.
- Prompt templates in `libs/prompts/<category>/<name>.txt`, laden via `load_prompt()`.

## Eval thresholds
Faithfulness ≥ 0.75, Relevance ≥ 0.70 — zie `tests/eval/test_regression.py`.
Drempel aanpassen vereist expliciete motivatie in de commit message.

## Lokaal starten
```bash
docker compose up qdrant -d
pip install -e ".[dev]"
cp .env.example .env  # vul API keys in
python scripts/ingest.py data/raw/
uvicorn apps.api.main:app --reload
```

## Testen
```bash
pytest tests/unit/                          # snel, geen deps
pytest tests/integration/ -m integration   # vereist geen draaiende services (mocks)
python scripts/run_eval.py                 # vereist Qdrant + LLM key
```

---
> Source: [AI-Savvy-NL/ai-platform](https://github.com/AI-Savvy-NL/ai-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
