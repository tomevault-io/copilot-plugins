## omniforge-ai

> - python -m venv .venv && source .venv/bin/activate

# Project instructions for Copilot

## How to run (minimum)
- Install:
  - python -m venv .venv && source .venv/bin/activate
  - pip install -r requirements.txt
- Run:
  - (fill) e.g. uvicorn app.main:app --reload
- Verify:
  - (fill) curl http://127.0.0.1:8000/health

## Project layout (what matters)
- app/: API entrypoints + routers
- services/: business logic
- configs/: config loading (.env)
- docs/: documents
- tests/: pytest

## Conventions
- Prefer small, incremental changes.
- Add logging for new flows.
- Add/adjust tests for behavior changes.

---
> Source: [ytt070666/OmniForge-AI](https://github.com/ytt070666/OmniForge-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
