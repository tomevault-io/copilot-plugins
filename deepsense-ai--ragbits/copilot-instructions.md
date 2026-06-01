## repository-setup

> This repository is using `uv` for package management. Rather than using `pip` use `uv pip` to install any packages and `uv run` to run python.


# Repository setup and pre-commit checks

This repository is using `uv` for package management. Rather than using `pip` use `uv pip` to install any packages and `uv run` to run python.

Run following checks after implementing any changes:

uv run ruff format
uv run ruff check --fix
uv run mypy <PATH>
pytest

---
> Source: [deepsense-ai/ragbits](https://github.com/deepsense-ai/ragbits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
