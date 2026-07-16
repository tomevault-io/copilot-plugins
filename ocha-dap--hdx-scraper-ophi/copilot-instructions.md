## hdx-scraper-ophi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**hdx-scraper-ophi** retrieves Multidimensional Poverty Index (MPI) data from the [Oxford Poverty and Human Development Initiative (OPHI)](https://ophi.org.uk/) and publishes it to HDX as per-country and global MPI datasets, and as a HAPI poverty rate dataset. It downloads Excel files from OPHI, a CSV from Google Sheets, and makes HDX reads and writes. It is run annually in late October (between the 20th and 24th, on the nearest weekday).

## Commands

Install dependencies:
```bash
uv sync
```

Run the scraper:
```bash
uv run python -m hdx.scraper.ophi
```

Run tests:
```bash
uv run pytest
```

Run a single test:
```bash
uv run pytest tests/test_main.py
```

Lint check:
```bash
pre-commit run --all-files
```

## Architecture

The pipeline in `__main__.py`:

1. **`Pipeline`** — Downloads and parses national, subnational, and trend Excel files from OPHI; matches admin-1 region names to P-codes using COD admin boundaries.
2. **`DatasetGenerator`** — Generates per-country and global MPI datasets and showcases for HDX.
3. **`HAPIOutput`** / **`HAPIDatasetGenerator`** — Derives a HAPI poverty rate dataset from the global MPI data and publishes it to HDX.

### Key design points

- **Config files**: Project configuration lives in `src/hdx/scraper/ophi/config/`.
- **Test fixtures**: Test input data lives in `tests/fixtures/input/`; expected output files live in `tests/fixtures/`.

## Environment

Requires `~/.hdx_configuration.yaml` with HDX credentials, or env vars: `HDX_KEY`, `HDX_SITE`, `USER_AGENT`, `TEMP_DIR`, `LOG_FILE_ONLY`.

Requires `~/.useragents.yaml` with a `hdx-scraper-ophi` entry.

## Collaboration Style

- Be objective, not agreeable. Act as a partner, not a sycophant. Push back when you disagree, flag tradeoffs honestly, and don't sugarcoat problems.
- Keep explanations brief and to the point.
- Don't rely on recalled knowledge for facts that could be stale (API behaviour, library versions, external systems). Search or read the actual source first.

## Scope of Changes

When fixing a bug or addressing PR feedback, change only what is necessary to resolve the specific issue. Do not refactor surrounding code, rename variables, adjust formatting, or make improvements in the same commit unless they are directly required by the fix.

---
> Source: [OCHA-DAP/hdx-scraper-ophi](https://github.com/OCHA-DAP/hdx-scraper-ophi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
