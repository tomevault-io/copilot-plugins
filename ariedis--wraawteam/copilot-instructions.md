## wraawteam

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository tracks football match data for **multiple teams**. Each team has its own isolated directory under `teams/<team-slug>/`, containing raw Dribl exports (XLSX snapshots), coach-entered notes, and generated processed datasets. Reports and insights are scoped to a single team at a time.

## Commands

**Install dependencies:**
```bash
pip install -r requirements.txt
```

**Convert the latest Dribl snapshot to CSV (run from repo root):**
```bash
python scripts/dribl_matches_export_to_csv.py --team-dir teams/WRR-AAW
```

**Convert a specific snapshot:**
```bash
python scripts/dribl_matches_export_to_csv.py --team-dir teams/WRR-AAW --input teams/WRR-AAW/data/raw/dribl/matches_YYYY-MM-DD.xlsx
```

**Write to a custom output path:**
```bash
python scripts/dribl_matches_export_to_csv.py --team-dir teams/WRR-AAW --output path/to/output.csv
```

**Generate an insights report:**
```bash
python scripts/generate_insights.py --team-dir teams/WRR-AAW
```

**Validate coach notes:**
```bash
python scripts/validate_notes.py --team-dir teams/WRR-AAW
```

**Add a player:**
```bash
python scripts/add_player.py --team-dir teams/WRR-AAW --name "Alice Brooks" --position GK
```

## Architecture

### Directory structure

```
teams/
  <team-slug>/
    data/
      raw/dribl/          ← Dribl XLSX exports (never edited in place)
      processed/          ← Machine-generated matches.csv
    notes/                ← Manually maintained coach notes
    reports/              ← Generated reports
    secrets/              ← Gitignored; player_id_map.csv lives here
docs/                     ← Shared reference docs (tag dictionary, positions)
scripts/                  ← Shared scripts, all accept --team-dir
```

### Data flow

1. **Raw snapshots** — Dribl XLSX exports saved to `teams/<team-slug>/data/raw/dribl/matches_YYYY-MM-DD.xlsx` (never edited in place).
2. **Converter script** — `scripts/dribl_matches_export_to_csv.py --team-dir teams/<team-slug>` reads the latest snapshot (by date in filename, falling back to mtime), renames columns to snake_case, normalizes date/duration/score types, sorts rows, and writes `teams/<team-slug>/data/processed/matches.csv`.
3. **Coach notes** — `notes/match_notes.csv` (one row per fixture) and `notes/match_tags.csv` (multiple rows per fixture) are manually maintained and keyed by `fixture_id`.

### Primary key

`fixture_id` (the Dribl `Identifier` column) is the join key across all datasets within a team.

### GitHub Actions workflows

All workflows auto-discover teams by listing subdirectories of `teams/` and running a matrix job per team.

- **`validate-dribl-snapshots_Version2.yml`** — Runs on PRs/pushes when XLSX files change. Enforces filename pattern `matches_YYYY-MM-DD.xlsx` and max size of 5 MiB.
- **`build-matches-csv_Version2.yml`** — Runs on push to `main` when XLSX/script/requirements change. Auto-commits updated `teams/<team>/data/processed/matches.csv`.
- **`check-matches-csv-up-to-date_Version2.yml`** — Runs on PRs. Fails if `matches.csv` doesn't match what the script would generate. **Always commit the regenerated CSV before opening a PR.**
- **`validate-notes.yml`** — Runs on PRs/pushes when notes CSVs change. Cross-validates notes against matches.

### Adding a new team

1. Create the directory structure:
   ```bash
   mkdir -p teams/<new-slug>/data/raw/dribl teams/<new-slug>/data/processed teams/<new-slug>/notes teams/<new-slug>/reports
   ```
2. Add the initial Dribl export as `teams/<new-slug>/data/raw/dribl/matches_YYYY-MM-DD.xlsx`.
3. Run `python scripts/dribl_matches_export_to_csv.py --team-dir teams/<new-slug>`.
4. Commit — CI will automatically detect the new team directory and run matrix jobs for it.

### Workflow for adding a new match export

1. Save Dribl export as `teams/<team-slug>/data/raw/dribl/matches_YYYY-MM-DD.xlsx`.
2. Run `python scripts/dribl_matches_export_to_csv.py --team-dir teams/<team-slug>`.
3. Commit both the new XLSX snapshot and the updated `teams/<team-slug>/data/processed/matches.csv`.
4. Open a PR — recommended branch name: `add-dribl-snapshot-YYYY-MM-DD`.

## Data conventions

- Snapshot filenames **must** match `matches_YYYY-MM-DD.xlsx` (enforced by CI).
- `data/processed/matches.csv` is machine-generated — always regenerate via the script, never edit manually.
- Coach notes use `;` to separate multiple items in free-text fields.
- Tags in `notes/match_tags.csv` use `lower_snake_case` (e.g., `conceded_set_piece`, `press_successful`).
- `secrets/` is gitignored per team (`teams/*/secrets/`). Never commit player names to version control.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ariedis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
