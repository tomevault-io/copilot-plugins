## sem-search

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands
- Run app: `python semantic_search.py`
- Build index: `python semantic_search.py --index --repo /path/to/java/repo`
- Search: `python semantic_search.py --search "query" --top-k 5`
- Lint: `flake8 semantic_search.py`
- Type check: `mypy semantic_search.py`
- Test: `pytest tests/`

## Code Style
- Imports: Group standard library, third-party, then local imports
- Types: Use type hints for function params and returns with `typing` module
- Naming: snake_case for variables/functions, PascalCase for classes
- Error handling: Use try/except blocks with specific exceptions
- Docstrings: Not currently used but follow Google style if adding
- Line length: Max 100 characters
- Use dataclasses for data container classes
- Favor composition over inheritance

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ignatov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ignatov)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
