## magi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Setup
uv venv && uv pip install -e ".[dev,web]"

# Run tests
.venv/bin/python -m pytest tests/ -v

# Run single test
.venv/bin/python -m pytest tests/test_vote.py -v

# Run CLI
.venv/bin/magi ask "question" --mode adaptive
.venv/bin/magi diff --staged
.venv/bin/magi dashboard --port 3000

# Build and publish
.venv/bin/python -m build
.venv/bin/python -m twine upload dist/*
```

## Architecture

MAGI is a structured disagreement engine. Three LLMs answer the same question independently, then coordinate via protocols to produce a Decision Dossier.

**Core flow:** `Engine.ask()` → parallel node queries → protocol (vote/critique/adaptive) → Decision

**Key design decisions:**
- All LLM calls go through `MagiNode.query()` which wraps LiteLLM. Reasoning models (e.g. MiniMax M2.7) may return content in `reasoning_content` instead of `content` — the node handles this.
- Vote mode uses structured position extraction (`POSITION: ...` tag) for real majority voting, not full-text comparison.
- Vote 3-way split (no majority) auto-escalates to critique mode.
- Critique mode (ICE) runs multi-round debate where models read and critique each other's answers. Agreement is measured by word-overlap heuristic (MVP — should be replaced with LLM-as-judge).
- Adaptive mode queries all 3 nodes first, computes agreement score, then routes: >0.8 → vote, 0.4-0.8 → critique, <0.4 → escalate.
- The NERV dashboard (`magi/web/`) uses FastAPI + WebSocket + vanilla HTML/JS. No React. The server streams events (`node_start`, `node_done`, `critique_start`, `decision`) to the frontend.

**Protocols are functions, not classes.** Each protocol (`vote()`, `critique()`, `adaptive()`) is an async function in `magi/protocols/` that takes a query and list of nodes, returns a Decision.

**Personas are configurable.** `magi/presets/__init__.py` defines 5 preset persona triples. The CLI `--preset` flag and the engine `personas` param control which perspectives the nodes use.

## Dependencies

- **litellm**: Unified LLM provider layer. Versions 1.82.7 and 1.82.8 are excluded (supply chain attack, Endor Labs 2026-03-24).
- **click**: CLI framework.
- **fastapi + uvicorn + websockets**: Optional, for `magi dashboard`. Install with `pip install magi-system[web]`.

## Testing

All tests use mocked LLM responses via `AsyncMock`. No real API calls in tests. pytest-asyncio with `mode=strict`.

## Branching

Use `feature/` branches for new features, `fix/` branches for bug fixes. Never commit directly to main.

---
> Source: [fshiori/magi](https://github.com/fshiori/magi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
