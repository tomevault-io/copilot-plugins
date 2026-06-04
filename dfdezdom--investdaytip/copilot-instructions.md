## investdaytip

> Purpose: concise guidance for Copilot sessions to understand and operate on InvestDayTip.

# Repository instructions for Copilot

Purpose: concise guidance for Copilot sessions to understand and operate on InvestDayTip.

## Current repository state

This repository contains a Python 3.10+ CLI application (InvestDayTip) that produces long-term stock & ETF recommendations and self-contained HTML reports. Source lives under src/investdaytip/ and unit tests are in tests/.

## Build, test, and lint commands

Prerequisites: Python 3.10+ and a virtual environment.

Installation (editable):

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -e .
# For development extras (tests):
pip install -e "[dev]"
```

Developer run (no install):

```bash
# Run CLI module directly for quick debugging or IDE runs
python -m investdaytip.main --help
# Or run installed entrypoint after editable install
investdaytip --help
```

Run the full test suite:

```bash
pytest -q
```

Run a single test function or file (examples):

```bash
# single test function (module::function)
pytest tests/test_scoring.py::test_score_stock -q
# single test file
pytest tests/test_scoring.py -q
# quick selection by keyword expression
pytest -q -k "test_score_stock or strong_stock"  # run tests matching expression
```


Preview generated HTML files locally:

```bash
./preview.sh
# then open http://localhost:8000/<generated-file>.html
```

`ruff` and `mypy` are configured in `pyproject.toml` (and installed via the `dev` extra). Lint with `ruff check src tests` and type-check with `mypy`. Follow PEP 8 and type hints (see CONTRIBUTING.md). Note: ruff's `UP` (pyupgrade) rule is intentionally disabled — keep `Optional[...]` for dataclass fields rather than `X | None`.

## High-level architecture (big picture)

- CLI (src/investdaytip/main.py): argument parsing and top-level output (rich table or export).
- Orchestration (src/investdaytip/recommender.py): builds universes, schedules concurrent fetches, aggregates results.
- Data ingestion (src/investdaytip/data_source.py): yfinance wrapper, dataclasses (StockData / EtfData). All network I/O is contained here.
- Scoring (src/investdaytip/scoring.py): pure functions mapping metrics → 0–100 sub-scores and composite score.
- Universes (src/investdaytip/*_universe.py): curated ticker lists (US, EU, Asia, ETFs). Naming convention: *_universe.py.
- Export (src/investdaytip/html_export.py): creates a self-contained HTML report with client-side filters and sorting.
- Tests (tests/): unit tests that exercise scoring and export logic without network access.

Data flow: CLI → recommender → data_source (yfinance) → scoring → html_export / CLI output.

## Key conventions (repository-specific)

- scoring.py must remain pure and side-effect free. Put any network or I/O code in data_source.py.
- New ticker universes must be a new module named *_universe.py and integrated via recommender._build_universe.
- recommend() and _build_universe() accept region and currency as str or list[str]. When region=="all" and currency is one or more specific codes, the region is derived from the currencies (USD→us, EUR→eu, JPY→asia) to reduce API calls.
- Tests must not perform live network calls — construct or mock StockData/EtfData directly.
- HTML exports are self-contained; filenames follow the `investDayTip[-<tag>]-yyyymmdd-hhmm.html` pattern used by the CLI.
- Ticker suffixes follow Yahoo Finance conventions; exchange mapping is used for Google Finance/TradingView links.
- Follow type hints and PEP 8 style; CONTRIBUTING.md documents these expectations for PRs.
- Use preview.sh to serve generated HTML for manual inspection.

## Files and references for reasoning

- README.md — usage, universes, export behavior, limitations
- CONTRIBUTING.md — contributor rules (pure scoring, tests, new universes)
- src/investdaytip/* — core modules (main, recommender, data_source, scoring, html_export)
- tests/* — unit tests and examples of test inputs without network
- tickers-files-examples/ — example ticker lists used by the CLI

## When to update

Update this file when: new build manifests or CI are added (pyproject.toml, setup.cfg, tox, GitHub Actions), linters are introduced, or the module layout/entrypoints change.

---

Updated from README.md and CONTRIBUTING.md to provide actionable guidance for Copilot sessions.

---
> Source: [dfdezdom/investdaytip](https://github.com/dfdezdom/investdaytip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
