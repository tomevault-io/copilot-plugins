## gngram-lookup

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make install              # poetry install
make test                 # poetry run pytest tests/ --no-header -rN
make lint                 # poetry run ruff check .
make all                  # install + ensure-data + ensure-pos-data + test

# Run a single test file
poetry run pytest tests/test_frequency.py --no-header -rN

# Run a single test by name
poetry run pytest tests/test_frequency.py -k "test_the" --no-header -rN

# Scripts (not distributed in wheel)
poetry run python scripts/cluster.py <word>
poetry run python scripts/erosion.py <word>
poetry run python scripts/suffix_fingerprint.py --min-tf 10000
poetry run python scripts/suffix_query.py --suffixes <suffix>
```

## Architecture

**Hash-bucketed O(1) lookup**: Words are stored across 256 parquet files (`00.parquet`–`ff.parquet`) keyed by MD5 hash prefix. Lookup hashes the word, reads one file, binary-searches for the hash suffix. Data lives at `~/.gngram-lookup/data/` (frequency) and `~/.gngram-lookup/pos-data/` (POS).

**Fallback chain** in `lookup.py`: direct lookup → contraction expansion ("don't" → "do") → possessive stripping ("ship's" → "ship") → hyphenated splitting ("quarter-deck" → parts).

**Two clustering strategies** in `lookup.py`:
- `prefix_cluster(word)` — flat list of words starting with the root string (forward expansion)
- `erosion_cluster(word)` — erodes suffixes to find the morphological stem, then expands laterally to find the full derivational family (returns nested tree)

**Word score**: Log-normalized 1–100 scale against "the" (most frequent). Compresses Zipfian distribution for meaningful resolution.

## Project Layout

- `gngram_lookup/` — distributed package (PyPI wheel). Core lookup, POS, CLI entry points, data downloaders.
- `scripts/` — analysis and exploration scripts (not in wheel). Output defaults to `/tmp/gngrams-lookup/`.
- `builder/` — data build pipeline from raw Google Books 1-gram TSVs (not in wheel).
- `tests/` — pytest suite. Requires data to be installed (`conftest.py` checks on startup).

## Code Conventions

- **Ruff** linter: line-length 120, ignores F403/F405/E402
- **Dependencies**: polars + pyarrow only (no NLTK — use `wordnet-lookup` sibling repo for WordNet membership)
- **Type hints** on function signatures
- **Do not commit** unless explicitly asked
- Public API is exported from `gngram_lookup/__init__.py`
- CLI entry points defined in `pyproject.toml` under `[tool.poetry.scripts]`: `score`, `exists`, `freq`, `pos`, `pos-freq`, `has-pos`

## Data Distribution

PyPI wheel (~50KB) contains only code. Frequency data (~110MB) and POS data are distributed as GitHub release tarballs, downloaded on first use via `python -m gngram_lookup.download_data`. Build pipeline: raw TSV → decade-aggregated parquet → hash-bucketed parquet → tar → GitHub release.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/craigtrim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/craigtrim)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
