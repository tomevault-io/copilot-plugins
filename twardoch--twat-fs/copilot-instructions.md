## twat-fs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**twat-fs** is a Python library providing unified file upload to multiple storage providers (anonymous and authenticated) with retry/fallback, async/sync support, and URL validation. It is a plugin for the `twat` ecosystem, registered as `twat.plugins.fs`.

## Build & Development Commands

```bash
# Install in dev mode
uv pip install -e ".[dev,test,all]"

# Run tests
uvx hatch test                          # default hatch test
pytest tests                            # direct pytest (verbose, durations, auto-asyncio)
pytest tests/test_upload.py::test_name  # single test

# Lint & format
hatch run fix                           # auto-fix + format
hatch run lint                          # check only
ruff check src/twat_fs tests            # direct ruff check
ruff format --respect-gitignore src/twat_fs tests

# Type checking
hatch run type-check                    # or: mypy src/twat_fs tests

# Test with coverage
hatch run test-cov

# Benchmarks
hatch run test:bench
```

## Architecture

### Provider System (core abstraction)

The upload system revolves around a **provider protocol** pattern:

1. **`upload_providers/protocols.py`** — Defines `Provider` and `ProviderClient` Protocols, plus `ProviderHelp` TypedDict. Every provider module must satisfy these protocols.

2. **Each provider module** (e.g., `catbox.py`, `s3.py`) exports four things:
   - `PROVIDER_HELP: ProviderHelp` — metadata dict (setup instructions, max_size, retention, auth_required)
   - `get_credentials() -> dict | None` — checks env vars, returns `None` if unavailable
   - `get_provider(credentials) -> ProviderClient` — instantiates the client
   - `upload_file(file_path, ...) -> str` — module-level upload function

3. **`upload_providers/factory.py`** — `ProviderFactory` singleton dynamically imports provider modules, validates they have required attributes, caches them.

4. **`upload_providers/simple.py`** — Base classes: `BaseProvider` (ABC), `AsyncBaseProvider` (wraps async→sync), `SyncBaseProvider` (wraps sync→async). Concrete providers subclass one of these.

5. **`upload_providers/core.py`** — Retry decorators (`@with_retry`, `@with_async_retry`), error types (`RetryableError` vs `NonRetryableError`), URL validation, timing metrics.

6. **`upload_providers/__init__.py`** — `PROVIDERS_PREFERENCE` list defines fallback order. Anonymous providers first (catbox, litterbox, x0at, ...), authenticated last (fal, s3, dropbox).

### Upload Orchestration

**`upload.py`** is the main upload entrypoint:
- `upload_file()` tries providers in `PROVIDERS_PREFERENCE` order with circular fallback
- Each provider gets retry attempts with exponential backoff
- `RetryableError` → retry same provider; `NonRetryableError` → skip to next
- `fragile=True` disables fallback (fail-fast mode)
- `UploadOptions` dataclass holds per-call config (remote_path, unique, force, fragile)

### Async/Sync Bridge

**`upload_providers/async_utils.py`** provides:
- `@to_sync` / `@to_async` decorators for wrapping functions either direction
- `run_async()` handles event loop detection and reuse
- `gather_with_concurrency()` for semaphore-limited parallel async ops

### CLI

**`cli.py`** uses `fire` for the CLI (`twat-fs` command). Key commands:
- `twat-fs upload FILE [--provider PROVIDER]` — upload a file
- `twat-fs upload_provider status [PROVIDER_ID] [--online]` — show provider status
- Provider list bracket notation: `--provider "[s3,dropbox,catbox]"`

### Registry & Metadata

**`upload_providers/registry.py`** loads service metadata from **`data/fileshare.toml`** — a TOML database with per-service rank, tier, status, max_size, retention, test results (timing, integrity verification).

### Exception Hierarchy

**`exceptions.py`** defines `TwatFsError` inheriting from `twat.common.exceptions.TwatError` (with a fallback if twat isn't installed).

## Provider Preference Order

Anonymous (no setup): catbox → litterbox → x0at → tmpfilelink → tmpfilesorg → senditsh → www0x0 → uguu → pixeldrain → filebin

Authenticated (require env vars): fal (`FAL_KEY`), s3 (`AWS_S3_BUCKET`, `AWS_DEFAULT_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`), dropbox (`DROPBOX_ACCESS_TOKEN`)

## Adding a New Provider

1. Create `src/twat_fs/upload_providers/<name>.py`
2. Subclass `AsyncBaseProvider` or `SyncBaseProvider` from `simple.py`
3. Export module-level: `PROVIDER_HELP`, `get_credentials()`, `get_provider()`, `upload_file()`
4. Add the provider name to `PROVIDERS_PREFERENCE` in `upload_providers/__init__.py`
5. Optionally add metadata to `data/fileshare.toml`

## Key Conventions

- Every source file has a `# this_file: <path>` comment near the top
- Python 3.10+ target (type hints use `X | Y` union syntax, not `Optional`)
- `loguru` for logging; level controlled by `LOGURU_LEVEL` env var
- Line length: 120 chars (ruff)
- pytest markers: `benchmark`, `unit`, `integration`, `slow`, `online`, `requires_api_key`
- `asyncio_mode = 'auto'` in pytest config — async tests run automatically
- Version is derived from git tags via `hatch-vcs` (no manual version file)

## Important Dependencies

- `aiohttp` — async HTTP for most anonymous providers
- `requests` — sync HTTP fallback
- `tenacity` — retry logic (used alongside custom retry decorators in `core.py`)
- `fire` — CLI framework
- `twat` — parent framework (provides base exceptions, plugin system)
- Optional: `boto3` (S3), `dropbox` (Dropbox SDK), `fal-client` (fal.ai)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twardoch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
