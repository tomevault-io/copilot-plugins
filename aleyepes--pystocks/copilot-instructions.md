## pystocks

> `pystocks` is an ETF factor analysis pipeline designed to calculate efficient frontier portfolios. It automates the lifecycle of financial data: from scraping product lists and fundamental data (via IBKR web proxies) to storage in a normalized SQLite database, preprocessing into analysis-ready features, and executing factor research.

# Gemini Project Context: pystocks

## Project Overview
`pystocks` is an ETF factor analysis pipeline designed to calculate efficient frontier portfolios. It automates the lifecycle of financial data: from scraping product lists and fundamental data (via IBKR web proxies) to storage in a normalized SQLite database, preprocessing into analysis-ready features, and executing factor research.

### Core Technologies
- Data Storage: SQLite (primary database at `data/pystocks.sqlite`)
- CLI Framework: [Python Fire](https://github.com/google/python-fire)
- Quality Tools: Ruff (linting/formatting), Pyright (static typing), Pytest (testing)
- Data Science Stack: pandas (heavy usage in preprocessing and analysis)

## Project Structure
- `pystocks/`: Active production codebase.
    - `ingest/`: Authentication (`session.py`), product scraping (`product_scraper.py`), and fundamentals scraping (`fundamentals.py`).
    - `storage/`: SQLite schema (`schema.py`), connection management (`_sqlite.py`), and normalization logic (`fundamentals_store.py`).
    - `preprocess/`: Clean and transform raw snapshots into point-in-time features (prices, dividends, snapshots).
    - `analysis/`: Research engine for analysis panels, factor returns, and ETF betas.
    - `tests/`: Mirrored structure of the runtime modules (e.g., `tests/ingest/`, `tests/storage/`).
- `docs/`: Operational notes, plans, and sample data (e.g., `sample_conids.txt`).
- `data/`: SQLite database and research/telemetry artifacts.
- `src/` & `notebooks/`: Historical/reference material; do not modify unless explicitly requested.

## Development Workflow

### Environment Setup
```bash
./venv/bin/pip install -r requirements.txt
./venv/bin/pre-commit install
```

### Key CLI Commands
Main entry point: `pystocks/cli.py`.
- Full Pipeline: `./venv/bin/python -m pystocks.cli run_pipeline --limit 100`
- Scraping: `./venv/bin/python -m pystocks.cli scrape_fundamentals --limit 100 --conids_file docs/sample_conids.txt`
- Preprocessing: `./venv/bin/python -m pystocks.cli preprocess_prices`
- Analysis: `./venv/bin/python -m pystocks.cli run_analysis`
- Maintenance: `./venv/bin/python -m pystocks.cli refresh_fundamentals_views` (Run after storage or view changes).

### Code Quality & Testing
Always run these before committing:
- Lint & Format: `./venv/bin/python -m ruff check . --fix && ./venv/bin/python -m ruff format .`
- Type Check: `./venv/bin/python -m pyright`
- Test: `./venv/bin/python -m pytest -q`

## Technical Conventions

### Coding Style
- Formatting: 4-space indentation, double quotes, snake_case for all symbols.
- Tooling: Let Ruff handle all formatting and import sorting.
- Typing: Pyright is enforced for `pystocks/`. New code must be fully typed.

### Testing Guidelines
- Use `pytest` and name files `test_*.py`.
- Add tests in the mirrored `pystocks/tests/` directory.
- Prioritize behavior-focused tests covering schema shape, effective-date rules, and pipeline wiring.

### Commit Guidelines
- Use conventional prefixes: `refactor:`, `build:`, `docs:`, `chore:`, `feat:`, `fix:`.
- Use imperative summary lines.

### Data & Architecture
- Storage Model: Snapshot-based. Raw payloads are hashed and stored in `raw_payload_blobs`; analysis should target normalized endpoint tables.
- Effective Dates: Anchored from `ratios.as_of_date`.
- Asynchronous I/O: Ingestion tasks are primarily `asyncio`-based.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AleYepes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AleYepes)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
