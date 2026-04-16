## gemini-game

> This document contains notes and observations from the Gemini CLI agent during the development of the Flappy Bird game.

## Gemini CLI Project Notes

This document contains notes and observations from the Gemini CLI agent during the development of the Flappy Bird game.

### `uv` Command Usage

- To add development dependencies, use `uv add --dev <package_name>`. For example: `uv add --dev ruff pytest mypy`.
- To run commands within the `uv` managed environment, use `uv run <command>`. For example: `uv run python -m gemini_game`.
- To sync dependencies after modifying `pyproject.toml`, use `uv sync`.

### Current Project Status

The project is set up with `uv`, `pygame` is installed, and development dependencies (`ruff`, `pytest`, `mypy`) are added. The `ruff` linter has been configured and run, and all linting issues have been resolved.

Currently, the game is not running as expected, with a `ModuleNotFoundError` when attempting to execute `uv run python -m gemini_game`. Debugging is ongoing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alanmpitts) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
