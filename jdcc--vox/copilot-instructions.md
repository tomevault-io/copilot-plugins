## vox

> - Use `uv` for environment management and running commands.

# AGENTS.md

## Tooling
- Use `uv` for environment management and running commands.
- Install deps with `uv sync` (creates/updates `.venv`).
- Run commands via `uv run ...` (e.g., `uv run vox ...`).

## Tests
- Default: `make test` (runs `uv run pytest --cov=src/vox --cov-branch --cov-report=term-missing`).
- Direct: `uv run pytest` (add `--cov` as needed).
- **No task is complete until all tests pass and there's 100% coverage.**

## Style
- Python 3.11+, source under `src/vox`.
- Ruff is configured (line length 100; lint E/F/I/W). Keep formatting consistent with existing files.
- Use type hints and docstrings as in current modules.

## Architecture
- Client/server split under `src/vox/client` and `src/vox/server`; keep responsibilities isolated.
- CLI lives in `src/vox/cli.py` and wires subcommands to module entry points.
- Config is centralized under `src/vox/config`; prefer loading via helpers instead of ad hoc env parsing.

## Code Shape
- Keep functions/classes small and single-purpose; prefer clear helpers over long multi-branch blocks.
- Avoid excessive inheritance; favor composition and explicit dependencies.
- Keep side effects at module boundaries (CLI, IO, network); keep core logic easy to test.

---
> Source: [jdcc/vox](https://github.com/jdcc/vox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
