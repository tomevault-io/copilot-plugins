## magazine-processing

> Compact guidance for OpenCode sessions working in this repo.

# AGENTS.md

Compact guidance for OpenCode sessions working in this repo.

## What this is

A Dockerized Python app that polls `/import` every 10 minutes for magazine PDFs, matches filenames against regex patterns in `config/magazines.yaml`, renames them with a standardized format, and moves them to `/processed/<Magazine Name>/`. Unrecognized files go to `/quarantine` and a report with YAML suggestions is generated there.

## Commands

```bash
# Build and run (production)
docker compose up --build

# Run locally (creates /import, /processed, /quarantine as absolute paths)
pip install -r requirements.txt
python -m app.main

# Run tests (loads the real magazines.yaml — broken patterns will fail)
python -m pytest tests/

# Bulk-rename existing files on Z: drive to match current templates
python scripts/rename_z_drive.py --dry-run   # preview
python scripts/rename_z_drive.py              # execute
```

## Architecture

- **`app/main.py`** — Entry point. Runs an infinite loop calling `process_existing()` every 10 minutes. Note: imports `PollingObserver` but does **not** use it; the loop is simple `time.sleep(600)`.
- **`app/processor.py`** — Core logic: loads `config/magazines.yaml`, compiles regexes, extracts dates, moves files. Also handles deletion for `delete: true` entries.
- **`app/quarantine_report.py`** — Scans `/quarantine`, groups unrecognized files by filename skeleton, and writes `/quarantine/quarantine_report.txt` with suggested YAML entries.
- **`app/version.py`** — Hardcoded version string. May drift from the `VERSION` file used by CI.
- **`scripts/rename_z_drive.py`** — Standalone utility that walks `/mnt/z`, matches PDFs against YAML patterns, and renames them in-place to match templates. Has hardcoded `FOLDER_RENAMES` and `SKIP_FOLDERS`.

## Key conventions

### `config/magazines.yaml`

- Keep entries **sorted alphabetically by `name`**.
- Use `delete: true` to auto-delete unwanted magazines (no `date_groups` or `template` needed).
- Use `pattern` for a single regex, or `patterns` for a list of variants (each with its own `date_groups`).
- `date_groups` values: `day`, `month`, `year`, `month_name`, `month_range`, `season`, `year_short`.
  - `year_short` → `2000 + value`
  - `season` → maps French/English season names to months (e.g. `hiver`/`winter` → 1)
  - `month_name` → parses English, French, and German month names and abbreviations (e.g. `janv`, `fév`, `avr`, `juil`, `aou`, `déc`, `oktober`)
  - `month_range` → parses first month from a range like `Mars-Avril`
  - `date_groups: []` → uses today's date
  - Omitting `day` defaults to 1; omitting `month` defaults to January.
- Template placeholders: `{name}`, `{date}` (YYYY-MM-DD), `{year}`, `{month}`, `{day}`, `{day_short}`, `{month_name}`, `{month_name_orig}`, `{month_range}`, `{original}`.
- Default template (if omitted): `{name} - {date}.pdf`.

### Filename normalization (happens before matching)

The processor silently rewrites filenames on disk before regex matching:
- Extension normalized to `.pdf`
- Unicode NFC normalization
- Curly apostrophes `'` → straight `'`
- Flag emoji prefixes stripped (e.g. `🇰🇼 `)
- `_fr.downmagaz.net` suffix stripped
- ` - TP` suffix stripped (case-insensitive)
- Browser duplicate suffixes `(1)`, ` (1)` stripped (or file deleted if original exists)

### Duplicate handling

- Exact destination filename already exists → incoming file is deleted.
- Same file size as an existing file in the destination folder (different name) → incoming file is deleted (near-duplicate detection).

### Environment / paths

When running locally, paths default to absolute `/import`, `/processed`, `/quarantine`. Override with:
- `IMPORT_DIR`
- `PROCESSED_DIR`
- `QUARANTINE_DIR`

In Docker, `docker-compose.yml` mounts `./config:/app/config` as a volume, so `config/magazines.yaml` changes take effect on container restart without rebuilding the image.

## CI / release

- `.github/workflows/build.yml` builds and pushes to `ghcr.io/vorkosigan76/magazine_processing` on every push to `main`.
- The workflow reads the version from the `VERSION` file. Keep `VERSION` in sync with `app/version.py` if you care about accurate version logging.

## Testing notes

- `tests/test_processor.py` loads the real `config/magazines.yaml`, so invalid regex or broken date parsing in new entries will fail the test suite.
- `tests/test_quarantine_report.py` tests the skeleton-based grouping and YAML suggestion logic.
- No lint, type-check, or formatter config exists in this repo; just run `pytest`.

## Quarantine workflow

When adding support for new magazines:
1. Drop sample PDFs into `/import`.
2. Let them move to `/quarantine` (unrecognized).
3. Read `/quarantine/quarantine_report.txt` for suggested YAML entries.
4. Copy the suggested entry into `config/magazines.yaml`, refine the regex and `date_groups` if needed, ensure alphabetical sort, and restart the container.
5. Re-run tests to confirm.

---
> Source: [Vorkosigan76/magazine_processing](https://github.com/Vorkosigan76/magazine_processing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
