## openspeclib

> OpenSpecLib is a Python toolkit that ingests spectral libraries (USGS Speclib 07, ECOSTRESS, RELAB, ASU TES, Bishop) into a unified data structure, validates it, and packages it for release. The catalog is JSON; spectral data is stored as one Apache Parquet file per source (zstd-compressed, row-grouped for partial Range-request reads) — category filtering is a column predicate, not a file selection.

# OpenSpecLib — Claude Code Guide

## Project Overview

OpenSpecLib is a Python toolkit that ingests spectral libraries (USGS Speclib 07, ECOSTRESS, RELAB, ASU TES, Bishop) into a unified data structure, validates it, and packages it for release. The catalog is JSON; spectral data is stored as one Apache Parquet file per source (zstd-compressed, row-grouped for partial Range-request reads) — category filtering is a column predicate, not a file selection.

## Repository Structure

- `src/openspeclib/models.py` — Pydantic models defining the in-memory data structure (single source of truth)
- `src/openspeclib/storage.py` — Parquet per-source file I/O (`ARROW_SCHEMA`, `write_source`, `read_chunk`, `iter_records`, `validate_parquet_schema`)
- `src/openspeclib/loaders/` — Per-source parsers (usgs.py, ecostress.py, relab.py, asu_tes.py, bishop.py)
- `src/openspeclib/combine.py` — Merges sources into master library (one Parquet per source + JSON catalog)
- `src/openspeclib/validate.py` — Schema + semantic validation
- `src/openspeclib/cli.py` — Click CLI (download, ingest, combine, validate)
- `schemas/` — `catalog.schema.json`, `spectrum.schema.json`, `library.arrow.schema.json` (generated), and `library.parquet-schema.md` (human-readable chunk reference)
- `tests/` — pytest tests with fixture data in `tests/fixtures/`
- `docs/` — Documentation (data-structure, pipeline, provenance, adding-sources)

## Key Commands

```bash
pip install -e ".[dev]"              # Install with all dependencies
pytest tests/ -v                     # Run tests
black src/ tests/ scripts/           # Format code
ruff check src/ tests/               # Lint (style, docstrings, type annotations)
python scripts/generate_schemas.py   # Regenerate JSON schemas
```

## Development Standards

### Formatting

- **Black** is the formatter. Line length is 100 characters.
- Run `black --check src/ tests/ scripts/` to verify; CI will reject unformatted code.

### Linting

- **Ruff** enforces pycodestyle (E/W), pyflakes (F), isort (I), pydocstyle (D), and flake8-annotations (ANN).
- Tests and scripts are exempt from docstring and annotation rules.

### Docstrings

- Use **Google-style** docstrings on all public and private functions/methods.
- Include `Args:`, `Returns:`, `Yields:`, and `Raises:` sections where applicable.
- Example:
  ```python
  def parse_file(filepath: Path) -> SpectrumRecord:
      """Parse a single spectrum file.

      Args:
          filepath: Path to the spectrum text file.

      Returns:
          A ``SpectrumRecord`` populated with parsed data.

      Raises:
          ValueError: If no spectral data is found.
      """
  ```

### Type Hints

- All function arguments and return values must have type annotations.
- Use `X | None` (Python 3.10+ union syntax) for optional types in non-Pydantic code.
- Pydantic models use `Optional[X]` with `Field()` to avoid annotation conflicts.

### CI

- CI runs on every push/PR to `main` via `.github/workflows/ci.yml`.
- Three jobs: **lint** (black + ruff), **typecheck** (mypy), **test** (pytest on 3.11 + 3.12).
- All checks must pass before merging.

## Conventions

- Pydantic models in `models.py` are authoritative — JSON schemas are derived from them
- Loaders yield `SpectrumRecord` via generators for memory efficiency
- Wavelengths are stored in native units (um, nm, cm-1) — never convert during ingest
- Source-specific metadata goes in `additional_properties`, not new schema fields
- All per-source Parquet file I/O goes through `openspeclib.storage` — never read or write them directly outside that module
- All tests, formatting, and lint checks should pass before any changes are committed

---
> Source: [null-jones/openspeclib](https://github.com/null-jones/openspeclib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
