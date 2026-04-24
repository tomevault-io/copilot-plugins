## caracal

> Caracal is a CLI tool for automated stock analysis. It fetches market data, persists it, and calculates technical indicators.

# CLAUDE.md — Caracal

## Project

Caracal is a CLI tool for automated stock analysis. It fetches market data, persists it, and calculates technical indicators.

## Tech Stack

- Python 3.12+, Click, Rich, yfinance, pandas, DuckDB
- Build: setuptools with flat layout
- Testing: pytest, ruff

## Commands

- `pip install -e ".[dev]"` — Install with dev dependencies
- `pytest` — Run tests
- `ruff check .` — Lint
- `ruff format .` — Format

## Conventions

- Code language: English (variables, comments, commits)
- Source layout: `caracal/` (flat layout) with subpackages (cli, providers, storage, indicators, analysis, output)
- Tests mirror source structure under `tests/`

## Security

For security reviews, use the Security Agent from the documentation repo (`~/Work/caracal/`):
- Command: `/security`
- Output: `SR-NNN` documents in `~/Work/caracal/documentation/security/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fuchsblau) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
