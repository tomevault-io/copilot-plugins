## systematically-improving-rag

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Educational course repository for "Systematically Improving RAG Applications" - teaches data-driven approaches to building and improving Retrieval-Augmented Generation (RAG) systems.

## Common Commands

```bash
# Package management (always use uv)
uv install                    # Install dependencies
uv add <package>              # Add new package
uv sync                       # Sync dependencies

# Code quality
uv run ruff check --fix --unsafe-fixes .
uv run ruff format .

# Documentation
mkdocs serve                  # Local dev server at localhost:8000
mkdocs build                  # Build static site

# Tests
uv run pytest -q              # Run tests
```

## Project Structure

```
latest/           # Current course version (weeks 0-6, case study)
cohort_1/         # Previous cohort materials (reference)
cohort_2/         # Previous cohort materials (reference)
docs/             # MkDocs documentation
  workshops/      # Main ebook content (chapters 0-7)
  office-hours/   # Q&A session summaries
  talks/          # Industry expert presentations
```

## Code Style

- Python 3.11, type hints throughout
- Async-first for I/O operations
- `typer` + `rich` for CLI tools
- `pydantic` for validation
- 9th-grade reading level for educational content
- No emojis in code or docs

## Key Patterns

- **RAG Flywheel**: Measure → Analyze → Improve → Iterate
- **Evaluation-first**: Always measure before and after changes
- **Synthetic data**: For cold-start problems

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jxnl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jxnl)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
