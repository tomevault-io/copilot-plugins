## upwork-analysis-project

> Core application code lives in `src/upwork_scraper/`. Use `cli.py` for entrypoints, `scraping.py` for HTML extraction, `processing.py` for record normalization, `config.py` for shared paths/settings, and `connectors/` for Airtable and Supabase integrations. Tests live in `tests/`. Operational inputs live in `config/`, SQL setup files in `database/schemas/`, helper scripts in `scripts/`, and exploratory work in `notebooks/`. Runtime data is written under `data/` and should stay out of commits unless intentionally adding fixtures.

# Repository Guidelines

## Project Structure & Module Organization
Core application code lives in `src/upwork_scraper/`. Use `cli.py` for entrypoints, `scraping.py` for HTML extraction, `processing.py` for record normalization, `config.py` for shared paths/settings, and `connectors/` for Airtable and Supabase integrations. Tests live in `tests/`. Operational inputs live in `config/`, SQL setup files in `database/schemas/`, helper scripts in `scripts/`, and exploratory work in `notebooks/`. Runtime data is written under `data/` and should stay out of commits unless intentionally adding fixtures.

## Build, Test, and Development Commands
Create the local environment with `uv sync`; this creates `.venv` from `pyproject.toml`. Install browser binaries once with `uv run playwright install`. Run the full workflow with `uv run python -m src.upwork_scraper.cli run-all --sync True`. Useful subcommands:

- `uv run python -m src.upwork_scraper.cli open-urls` opens saved search URLs in Firefox.
- `uv run python -m src.upwork_scraper.cli sync-airtable` pushes current records to Airtable.
- `uv run python -m src.upwork_scraper.cli cleanup` removes generated HTML/JSON artifacts.
- `uv run pytest` runs the test suite with strict marker handling from `pytest.ini`.

## Coding Style & Naming Conventions
Follow existing Python style: 4-space indentation, `snake_case` for functions/modules, `UPPER_SNAKE_CASE` for constants, and concise docstrings on non-trivial functions. Keep package imports relative inside `src/upwork_scraper`. Type hints are already used in the main package and should be preserved for new or edited functions. No formatter config is committed yet, so match the surrounding file style and keep changes minimal.

## Testing Guidelines
Use `pytest` for both unit and integration coverage. Name tests `test_*.py` and keep them close to the module behavior they verify. Prefer pure unit tests for parsing and transformation logic; isolate Supabase/Airtable integration coverage behind fixtures or explicit markers when adding new tests. If a change affects the ETL flow, validate both the narrow function and the CLI path it feeds.

## Commit & Pull Request Guidelines
Recent commits use short, imperative, lower-case summaries such as `prepared project for sharing` and `tidied more code`. Keep that style: one focused change per commit, clear subject line, no prefix noise unless the repo adopts one later. PRs should explain the workflow impact, note any config or schema changes, link the related issue if one exists, and include screenshots only when documentation or Airtable-facing behavior changes.

## Security & Configuration Tips
Do not commit populated `.env` files or service credentials. Treat `config/search_urls.yml`, Airtable schema files, and SQL migrations as reviewed configuration changes because they directly affect scraping targets and downstream storage.

---
> Source: [joshslade/upwork-analysis-project](https://github.com/joshslade/upwork-analysis-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
