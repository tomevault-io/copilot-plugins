## decompose

> Inherits from `../CLAUDE.md`. Read that first.

# decompose — Project Instructions

Inherits from `../CLAUDE.md`. Read that first.

## What This Is

Decompose is the deterministic foundation of the echology system. It classifies any text into structured semantic units — authority, risk, attention, entities — without an LLM. No probability. No hallucination. No cost.

**This is the most important project in echology.** Every other system depends on it. Changes here propagate everywhere.

## Architecture

Five modules, one pipeline:

```
text → chunker → classifier → entities → irreducibility → DecomposeResult
```

| Module | What It Does |
|---|---|
| `chunker.py` | Semantic chunking, Markdown-aware, sentence-boundary splitting |
| `classifier.py` | Regex-based authority + risk + content type classification. Attention scoring. |
| `entities.py` | Regex entity extraction: standards, dates, financial values, legal references |
| `irreducibility.py` | Detects content that must be preserved verbatim (specs, limits, formulas) |
| `core.py` | Orchestration. `decompose_text()` and `filter_for_llm()` |

## Interfaces

- **Python library:** `from decompose import decompose_text, filter_for_llm`
- **CLI:** `decompose --text "..." --pretty` or piped stdin
- **MCP server:** `decompose-mcp --serve` (exposes `decompose_text` and `decompose_url` tools)

## Rules

- **Zero runtime dependencies.** The library itself imports nothing outside the standard library. `mcp` is only required for MCP server mode. Do not add dependencies.
- **Deterministic.** Same input always produces same output. No randomness. No LLM calls. No network calls (except `decompose_url` which fetches the URL, then classifies deterministically).
- **Published on PyPI** as `decompose-mcp`. Changes must not break the public API (`decompose_text`, `filter_for_llm`, `DecomposeResult`, `Unit`).
- **63 tests.** Run `pytest` before any commit to a core module. Do not reduce test coverage.
- **Regex patterns are the core IP.** When modifying classifier patterns, test against real documents from multiple domains (AEC, insurance, legal, general). A pattern that improves one domain but breaks another is rejected.

## Key Files

- `src/decompose/core.py` — entry point, `decompose_text()`, `filter_for_llm()`
- `src/decompose/classifier.py` — authority/risk/content patterns and scoring
- `src/decompose/entities.py` — entity extraction patterns
- `src/decompose/irreducibility.py` — verbatim preservation detection
- `src/decompose/chunker.py` — semantic text chunking
- `src/decompose/mcp_server.py` — MCP server implementation
- `src/decompose/cli.py` — CLI entry point
- `tests/` — 63 tests across 6 modules

## Development

```bash
cd ~/echology/decompose
source .venv/bin/activate
pytest                    # run tests
ruff check src/ tests/    # lint
ruff format src/ tests/   # format
```

---
> Source: [echology-io/decompose](https://github.com/echology-io/decompose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
