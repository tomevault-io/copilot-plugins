## claude-test1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Run all tests (includes coverage)
pipenv run pytest

# Run single test file
pipenv run pytest tests/test_recovery_points.py

# Run single test
pipenv run pytest tests/test_recovery_points.py::test_save_recovery_points_to_file

# Unit tests only (mocked, fast)
pipenv run pytest -m unit

# Integration tests (requires AWS credentials)
RUN_INTEGRATION_TESTS=1 pipenv run pytest -m integration

# Run the app
python src/main.py
```

## Architecture

AWS Backup recovery points exporter using boto3. Core module is `src/recovery_points.py`:

- `list_all_recovery_points()` - Paginated listing from backup vault
- `get_recovery_point_tags()` - Retrieves tags per recovery point
- `fetch_recovery_points_with_tags()` - Combines listing + tags enrichment
- `export_recovery_points()` - Main entry point, outputs to `./data/recovery_points.json`

## Testing

Uses moto for AWS mocking. Some AWS Backup operations aren't fully supported by moto - tests skip gracefully with `pytest.skip()` when hitting "Not yet implemented" errors.

Mark tests with `@pytest.mark.unit` or `@pytest.mark.integration`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jayers99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
