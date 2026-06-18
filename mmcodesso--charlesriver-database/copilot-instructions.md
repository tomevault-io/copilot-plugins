## charlesriver-database

> `greenfield_database` is the source repository for the Charles River synthetic accounting analytics teaching environment. The primary artifact is the Docusaurus documentation site; the SQLite/Excel/support-workbook/CSV outputs and generated report assets exist to support that site and classroom use.

# AGENTS.md

## Project Overview

`greenfield_database` is the source repository for the Charles River synthetic accounting analytics teaching environment. The primary artifact is the Docusaurus documentation site; the SQLite/Excel/support-workbook/CSV outputs and generated report assets exist to support that site and classroom use.

The default package is a student-facing, anomaly-enriched teaching build. Separate clean, validation, and performance configs exist for reconciliation work and smaller local runs.

## Repository Map

- `docs/`: authored student/instructor docs. Files are Markdown with YAML front matter, but many pages use MDX imports/components inside `.md`.
- `queries/`: curated SQL packs for financial, managerial, audit, and case work. These files are also served directly by Docusaurus as static assets.
- `config/`: source-of-truth settings and catalogs. Key files are `settings*.yaml`, `anomaly_profile.yaml`, `accounts.csv`, `report_catalog.yaml`, and `report_pack_catalog.yaml`.
- `src/generator_dataset/`: Python dataset generator, exporters, settings, validations, and schema registry. `schema.py` is the canonical table/column definition source.
- `src/components/`, `src/pages/`, `src/theme/`, `src/css/`: Docusaurus UI code and custom MDX components.
- `src/generated/queryManifest.js`, `src/generated/reportManifest.js`, `src/generated/reportDocCollections.js`, `src/generated/reportPackManifest.js`: script-generated site data. Regenerate; do not hand-edit.
- `src/generated/queryDocCollections.js`: despite the folder name, no generator script is present in this repo. Treat it as curated source that must be kept in sync manually with query/docs changes.
- `scripts/`: Node scripts for branding/manifests/report-asset prep plus `scripts/generate-site-reports.py` for report export from an existing SQLite file.
- `static/`: site assets. `static/CNAME`, `static/img/favicon.svg`, and `static/img/site-social-card.svg` are generated from branding settings. `static/reports/` holds generated report preview/download assets and is ignored.
- `outputs/`: local generated datasets, reports, and logs. Ignored.
- `tests/`: pytest coverage for generator behavior, SQL semantics, report exports, docs structure, and teaching-flow invariants.
- `.github/workflows/deploy-docs.yml`: GitHub Pages build/deploy workflow.

## Setup And Commands

- Local prerequisites:
  - Python is required for dataset generation/tests. CI uses Python `3.13`.
  - Node `>=20` is required locally. CI uses Node `24`.
- Python environment:
  ```bash
  python -m venv .venv
  # activate the environment for your shell
  pip install -r requirements.txt
  ```
- For all agent-driven Python commands, use the repo-local virtual environment instead of the system interpreter. Prefer invoking the interpreter directly, for example:
  ```bash
  .venv\Scripts\python.exe -m pip install -r requirements.txt
  .venv\Scripts\python.exe -m pytest -q
  .venv\Scripts\python.exe -B -m compileall -q src tests
  ```
  On non-Windows shells, use `.venv/bin/python`.
- Node dependencies:
  ```bash
  npm ci
  ```
- Docusaurus development server:
  ```bash
  npm run start
  ```
- Production site build:
  ```bash
  npm run build
  ```
- Serve the built site:
  ```bash
  npm run serve
  ```
- Clear Docusaurus cache:
  ```bash
  npm run clear
  ```
- Individual manifest/branding/report-asset commands:
  ```bash
  npm run generate-site-branding
  npm run generate-query-manifest
  npm run generate-report-manifest
  npm run generate-report-pack-manifest
  npm run prepare-report-assets
  npm run validate-report-assets
  ```
- Dataset generation:
  ```bash
  .venv\Scripts\python.exe generate_dataset.py
  .venv\Scripts\python.exe generate_dataset.py config/settings_validation.yaml core
  ```
- Report asset generation from an existing SQLite file:
  ```bash
  .venv\Scripts\python.exe scripts/generate-site-reports.py --config config/settings.yaml --sqlite-path outputs/CharlesRiver.sqlite --report-output-dir static/reports
  ```
- Tests and Python syntax sanity:
  ```bash
  .venv\Scripts\python.exe -m pytest -q
  .venv\Scripts\python.exe -B -m compileall -q src tests
  ```
- Any agent- or tool-driven test invocation must use a timeout that matches the scope. Use at least one hour (`3600000` ms) for small targeted slices, and at least two hours (`7200000` ms) for broad multi-module runs or full-suite validation.
- Package deploy command:
  ```bash
  npm run deploy
  ```

Notes:

- `npm run start`, `npm run build`, and `npm run serve` already run the pre-hooks that generate branding/manifests and prepare/validate report assets.
- If `static/reports/` is missing, `npm run start` and `npm run build` may try to download the published SQLite asset using `published_sqlite_url` in `config/settings.yaml` or `REPORTS_SQLITE_URL`.
- In this Windows PowerShell workspace, prefer search/read commands that are stable here: use `git grep -n -- "<text>"` for tracked-text search, `Select-String -Path <file> -SimpleMatch -Pattern '<literal>'` for literal file search, and `Get-Content <file>` or `Get-Content <file> | Select-Object -Index ...` for targeted line reads.
- If `rg` is unavailable, returns `Access is denied`, or starts requiring awkward escaping, switch immediately to `git grep`, `Select-String`, and `Get-Content` instead of retrying `rg`.
- When running PowerShell commands with search patterns or inline text, prefer single-quoted literals and intermediate variables or here-strings for complex text so patches stay targeted and quoting failures do not derail the edit flow.
- No lint or formatter script is configured. Do not invent one in session handoffs.

## Coding And Editing Conventions

- Python follows the existing style: 4-space indentation, type hints, `pathlib`, dataclasses/settings objects, and pandas-driven table construction.
- Site code is plain JavaScript/JSX, not TypeScript. Preserve the existing 2-space indentation and component structure.
- Keep edits narrow and reviewable. Stable file names, query names, section headings, slugs, and routes matter because docs, manifests, and tests reference them directly.
- On PowerShell, avoid unnecessarily complex one-line command quoting. Prefer shorter native commands, intermediate variables, or separate reads/searches when locating edit points.
- Keep generated outputs out of Git. Do not hand-edit `build/`, `.docusaurus/`, `static/reports/`, `outputs/`, or the script-generated files in `src/generated/`.
- If branding changes, update the source settings/helper and rerun `npm run generate-site-branding`. Do not hand-edit `static/CNAME` or the generated SVGs.
- If query, report, or report-pack inventories change, update the source SQL/YAML/docs first and regenerate the relevant manifests.
- Preserve deterministic generation for a fixed `random_seed`.
- Preserve voucher-level `GLEntry` balance and statement/query tie-out behavior.
- New anomaly types must be reflected in `config/anomaly_profile.yaml`, the relevant docs, and tests.

## Documentation Guidance

- This is a docs-first Docusaurus repo. Author pages in `docs/**/*.md` with YAML front matter; MDX imports/components inside `.md` are normal here.
- Keep internal links as relative `.md` links, matching the existing docs style. `docusaurus.config.js` is configured to throw on broken links and broken markdown links.
- Preserve the teaching sequence: business context first, then process flow, then analytics/reports/cases. Do not rewrite student-facing pages into generic feature or product marketing copy.
- Public docs should not reintroduce the template phrases already blocked by tests, including `Audience and Purpose`, `Use this page when`, `Use this case when`, `Use this pack`, `Use this perspective`, and `Why this matters`.
- Case pages are expected to keep `## Key Data Sources`, `## Recommended Query Sequence`, and `## Next Steps`. Existing cases use `QuerySequence` from `@site/src/components/QueryReference`.
- Report-library pages use `ReportCatalog` plus `reportAreaCollections`. Report-perspective pages use `ReportLearningPack` plus `reportPackManifest`.
- Shared naming/file copy should use the site-branding components from `src/components/SiteBranding` such as `<DisplayName />`, `<DatasetName />`, `<FileName />`, and related helpers instead of hard-coding names.
- Mermaid is enabled. Keep diagrams simple, readable, and faithful to process/schema logic.
- If you change a page slug or route, also update `sidebars.js` and any redirect rules in `docusaurus.config.js`.

## Data, SQL, And Analytics Guidance

- There is no raw external data directory. Source-of-truth lives in Python code plus `config/*.yaml` and `config/accounts.csv`.
- `src/generator_dataset/schema.py` is the canonical schema registry. Update it first when tables or columns change, then update docs, queries, and tests.
- `queries/` is both teaching content and a public static asset directory. Renaming a SQL file changes docs references, generated manifests, and public URLs.
- `src/generated/queryManifest.js` is derived from the `queries/` tree. Regenerate it after adding, removing, or renaming SQL files.
- `src/generated/queryDocCollections.js` drives starter query maps, case sequences, and audit coverage text. Keep it synchronized manually when query coverage or teaching paths change.
- `config/report_catalog.yaml` is the source of truth for report definitions and asset paths.
- `config/report_pack_catalog.yaml` is the source of truth for guided report perspectives and their recommended sequences.
- Default teaching build: `config/settings.yaml` with `anomaly_mode: standard`, covering fiscal years 2024 through 2026.
- Clean reconciliation build: `config/settings_reconciliation.yaml` with `anomaly_mode: none` and separate clean SQLite/log outputs.
- Small validation and performance configs: `config/settings_validation.yaml` and `config/settings_perf.yaml`.
- Report assets under `static/reports/` or `outputs/reports/` are generated from SQLite and must never be hand-edited.
- Do not hand-edit `.sqlite`, `.xlsx`, `.zip`, `.csv`, or `preview.json` outputs. Regenerate them from code/config.
- Support-workbook validation sheets are the current validation companion. `export_validation_report()` is now a deprecated no-op, so do not depend on a standalone `outputs/validation_report.json`.
- Do not casually delete or overwrite `outputs/CharlesRiver.sqlite` or `outputs/CharlesRiver_clean.sqlite`; some published-build tests expect a populated SQLite file in `outputs/`.

## Validation Rules

- Before finishing, run the checks that match the touched area and report what actually ran.
- For Python validation or dataset-generation commands, use the interpreter from the repo-local `.venv` so tests run against the project environment, not the system Python.
- Do not default to one monolithic `pytest -q` run when the touched area can be validated in smaller slices. Prefer targeted module-level or feature-level pytest commands first, then add broader slices only when needed.
- For explicit test runs, split validation into smaller commands whenever practical so failures surface sooner and long-running groups do not block the whole validation pass.
- Use command timeouts that match the slice size:
  - at least one hour (`3600000` ms) for small targeted pytest slices
  - at least two hours (`7200000` ms) for broad multi-module slices, `pytest -q --durations=30`, or full-suite runs
- Minimum for Python, SQL, config, schema, exporter, or generator changes:
  ```bash
  .venv\Scripts\python.exe -m pytest -q <smallest relevant test slice>
  .venv\Scripts\python.exe -B -m compileall -q src tests
  ```
- If broad Python regression coverage is still required after targeted checks, run it as multiple pytest slices when possible, for example by affected module groups or test files, rather than assuming the entire suite should run in one command.
- Minimum for docs, Docusaurus config, sidebar, component, query-catalog, report-catalog, or branding changes:
  ```bash
  npm run build
  ```
- When changing query files, report catalogs, report packs, or branding inputs, make sure the generated manifests/assets were refreshed. A full `npm run build` already triggers the prebuild generators.
- When changing dataset generation behavior, settings, anomalies, schema, or report exports, also run the relevant dataset build such as:
  ```bash
  .venv\Scripts\python.exe generate_dataset.py
  ```
  or the smallest relevant alternate-config run through the same `.venv` interpreter.
- Manual checks still matter:
  - confirm the docs still read in the intended process-led sequence
  - confirm report preview/download cards still point to existing assets
  - confirm route or file changes did not break sidebar entries or internal links
- A change is not complete if it updates queries, docs, or catalogs without syncing the corresponding manifests/tests.

## Change Management

- Preserve the project's pedagogical structure and stable navigation. Prefer additive edits over reorganizing sections, slugs, or query names.
- Avoid large refactors in the generator unless the task requires them. Many tests assert exact teaching flows, section labels, query inventories, and report inventories.
- Avoid introducing new Python or npm dependencies unless the task clearly requires them and the tradeoff is documented.
- Keep generated files and heavyweight local outputs out of commits unless the repo already tracks them intentionally.
- When a source change implies generated follow-on files, regenerate only the affected artifacts and keep the diff reviewable.

## Handoff

At the end of a session, summarize:

- which files changed
- which commands ran
- validation results, including anything not run
- whether any manifests or generated assets were refreshed
- unresolved issues, assumptions, or known doc/code drift

If the change touched teaching copy, say whether the process-led flow, case-section headings, and query/report links were preserved.

If the change touched dataset or query logic, say which settings file or build variant you validated against: `config/settings.yaml`, `config/settings_reconciliation.yaml`, `config/settings_validation.yaml`, or `config/settings_perf.yaml`.

## Candidate Skills

- Refreshing Docusaurus manifests and report assets after SQL/catalog/branding changes.
- Running clean-vs-anomaly reconciliation workflows for statement or query debugging.
- Applying the docs editorial guardrails for case pages, process-led copy, and sidebar/link maintenance.

---
> Source: [mmcodesso/CharlesRiver_Database](https://github.com/mmcodesso/CharlesRiver_Database) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
