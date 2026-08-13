## reports-generation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment

All Python scripts must run under `~/venv`. The scripts self-reinvoke under the venv automatically, but for direct execution:

```bash
~/venv/bin/python generate_reports.py <metric_csv>
~/venv/bin/python parse_metrics_csv.py <metric_csv> [brand_name]
~/venv/bin/python build_heytruffle_reports.py <metric_csv> [--brands "..."] [--trend-csvs "..."]
~/venv/bin/python upload_to_drive.py [--month January_2026]
```

`build_heytruffle_reports.py` also shells out to `node generate_report.js` — `npm install` must have been run once (see `package.json`).

## Two report pipelines

There are currently **two generators** in this repo, both driven by the same Metabase CSV export:

1. **RestoHost pipeline** (original, `generate_reports.py`) — Python-only, fills `template.pptx` directly via `python-pptx`. See "Running Reports" below.
2. **heytruffle pipeline** (new, `build_heytruffle_reports.py`) — the rebrand target. Design is decoupled from data: a Node/`pptxgenjs` generator draws the "Monthly Impact Report" layout from scratch and reads all values from a per-brand JSON file. See "heytruffle Pipeline" below.

The heytruffle pipeline is intended to replace the RestoHost one once all clients are onboarded (`clients_config.json` populated + curated insights written for each). Until then, treat the RestoHost pipeline as the still-in-use default for any brand not yet configured in `clients_config.json`.

## Running Reports (RestoHost pipeline)

```bash
# All brands from a single CSV
~/venv/bin/python generate_reports.py /path/to/exported.csv

# Specific brands only
~/venv/bin/python generate_reports.py /path/to/exported.csv --brands "Felino,KYU"

# With trend chart (ordered oldest→newest CSVs; only applies to brands in --brands)
~/venv/bin/python generate_reports.py /path/to/exported.csv \
  --brands "Rreal Tacos" \
  --trend-csvs "nov.csv,dec.csv,jan.csv"

# Debug a single brand's metrics without generating PPTX
~/venv/bin/python parse_metrics_csv.py /path/to/exported.csv "Brand Name"
```

## Running Reports (heytruffle pipeline)

```bash
# One brand, with a 3-month trend for the "calls handled" chart
~/venv/bin/python build_heytruffle_reports.py /path/to/may.csv \
  --trend-csvs "mar.csv,apr.csv,may.csv" --brands "Aplos Mediterranean"

# All brands present in clients_config.json (default when --brands is omitted)
~/venv/bin/python build_heytruffle_reports.py /path/to/exported.csv
```

Per-brand steps performed by `build_heytruffle_reports.py`:
1. Load that brand's rows from the month CSV (skips the brand if absent).
2. Look up the brand in `clients_config.json` (skips the brand if no entry — see below).
3. `report_auto.compute_auto()` — reuses `parse_metrics_csv.stream_brand_metrics()` for `recovered_count`/`roi_hours`, and separately computes `assisted_revenue` from `smsByCategory` "guest action" categories (pickup, waitlist, catering, reservation, delivery, private events): `revenue = guest_actions × 80% conversion × guests × avg_ticket`. It also computes the "Main Insights" block (`AI Resolution Rate` and `Calls Handled`, each compared against the previous month if `--trend-csvs` has a prior-month entry) and a `has_trend_history` flag — `false` when fewer than 3 months of trend data exist, in which case `generate_report.js` renders a "Solved by AI" resolution chart instead of the (otherwise sparse) calls-handled bar chart.
4. Load `data/curated/<Brand>_<YYYY-MM>.json` (hand-written `listen` block only — two real call reasons; everything else in the report is auto-computed). If missing, `TODO` placeholders are used and a warning is printed — the report will still generate but needs a manual pass before sending.
5. Merge `client` (from `clients_config.json`) + `period` + `auto` + `curated` into `data/generated/<Brand>_<YYYY-MM>.json`.
6. Run `node generate_report.js <data.json> <out.pptx>` to render the PPTX (`pptx/<Brand>_Report_<Month><Year>.pptx`).

**`clients_config.json`** (gitignored — real client data, never committed) holds per-brand `name`, `tagline`, `locations_line`, `logo`, `guests` (avg party size), and `avg_ticket`. `clients_config.example.json` (committed) shows the expected shape — copy it to `clients_config.json` and fill in real values for each brand. `avg_ticket` currently defaults to a flat **$25** per brand (Google Places-based estimation is not wired up yet, see `HEYTRUFFLE_REBRAND.md`); override per-brand once real data exists — `Aplos Mediterranean` already has a validated value ($20) and should not be changed without re-checking against real data.

**`data/curated/`** and **`data/generated/`** are gitignored — they contain real client metrics and hand-written call-highlight data and must never be committed.

## Architecture

The pipeline is single-pass and sequential per brand:

**`generate_reports.py`** — orchestrator. Loads the CSV once via `load_all_brand_rows()`, then for each brand: aggregates metrics → generates chart → builds PPTX → accumulates summary.

**`parse_metrics_csv.py`** — all metric logic lives here. `stream_brand_metrics()` takes pre-loaded rows and returns a flat `metrics` dict. Insights are computed deterministically inside `_compute_insights()` — no AI calls. Key formulas:
- ROI: `solved_minutes * 2 / 60 * $18` (solved_minutes = Solved/total × total_minutes)
- Recovered: outside-hours calls (from `callsPerDay`) + `simultaneousCalls`

**`chart_generator.py`** — two chart types: `generate_chart()` (donut: Solved/Partial/ByPass) and `generate_trend_chart()` (bar: total calls per month). Chart type per brand is determined by `client_strategies.json`.

**`pptx_builder.py`** — uses only slide 0 from `template.pptx`. Replaces text placeholders (e.g. `[CLIENT_NAME]`, `[TOTAL_CALLS]`) and swaps the `[TOTAL_CALLS_EVOLUTION_GRAPH]` shape with the chart PNG. **Chart must be inserted before text replacement** (current order in code is intentional).

**`upload_to_drive.py`** — outputs a JSON manifest of files in `pptx/`. The actual upload is performed by Claude using Google Drive MCP tools. Parent Drive folder ID: `1ShpBt9HgN7e-wy8hnUIu3YeUfpKpRkLr`.

**`client_strategies.json`** — per-brand config for chart type (`trend` vs `resolution_donut`) and focus areas. Currently informational only — chart type selection in `generate_reports.py` is driven by whether `--trend-csvs` + `--brands` flags are passed, not by this file.

### heytruffle pipeline files

**`build_heytruffle_reports.py`** — orchestrator described under "Running Reports (heytruffle pipeline)" above.

**`report_auto.py`** — computes the `auto` data block: reuses `parse_metrics_csv.stream_brand_metrics()` for `recovered_count`/`roi_hours`, and adds `assisted_revenue`/`captured` (guest-action SMS breakdown) — all deterministic, no AI calls.

**`generate_report.js`** — renders the "Monthly Impact Report" PPTX (A4 **landscape**) from a merged data JSON using `pptxgenjs`. Layout: a full-width revenue card on top, then 3 columns below — Highlights of the Month (chart + recovered/time-off stats), Main Insights, and Listen to a Real Call. The Highlights chart is a bar chart for brands with 3+ months of `--trend-csvs` data, or — for newer brands (`has_trend_history: false`) — a resolution breakdown drawn as a **pie chart with a plain white circle over its center** to fake a donut look. (pptxgenjs doesn't support a fixed data-label position for its native `doughnut` chart type — without one, PowerPoint auto-shrinks each percentage label differently per slice. A real `pie` chart does support a fixed `"ctr"` label position, so all three labels render at the same size; don't "simplify" this back to `doughnut`.) Draws the full layout from brand tokens/fonts in `assets/` — does not use `template.pptx`. Requires the `Gowun Batang` and `Google Sans` fonts installed on the rendering machine.

## PPTX Template Placeholders

**RestoHost pipeline only** — the heytruffle pipeline does not use `template.pptx` or placeholder strings; it draws the layout directly in `generate_report.js` from the merged data JSON.

Exact strings expected in `template.pptx` slide 0:

| Placeholder | Value |
|---|---|
| `[CLIENT_NAME]` | Brand name |
| `[LAST_MONTH]` | Month name (e.g. "January") |
| `[CURRENT_YEAR]` | Year string |
| `[TOTAL_CALLS]` | Formatted integer |
| `[TOTAL_SMS_SENTt]  mainly for [MAIN_SMS_REASON` | Full SMS text (combined match) |
| `[% RECOVERED_CALLS]%` | e.g. "42.3%" |
| `[ROI_INISGHT]` | Insight 1 (note: typo in template) |
| `[CALL_TOPIC_INSIGH]` | Insight 2 (note: typo in template) |
| `[OTHER_INSIGHT]` | Insight 3 |
| `[TOTAL_CALLS_EVOLUTION_GRAPH]` | Replaced by chart image |

Placeholder typos (`[ROI_INISGHT]`, `[CALL_TOPIC_INSIGH]`, `[TOTAL_SMS_SENTt]`) match the actual template — do not fix them in code without updating the template.

## Output Locations

- `pptx/` — generated PPTX files (`BrandName_Report_MonthYYYY.pptx`), from either pipeline
- `charts/` — chart PNGs (`BrandName_chart.png` or `BrandName_trend_chart.png`) — RestoHost pipeline only
- `output/review_summary.md` — per-brand stats table with anomaly flags — RestoHost pipeline only
- `data/generated/<Brand>_<YYYY-MM>.json` — merged data JSON fed to `generate_report.js` — heytruffle pipeline only, gitignored (real client data)
- `data/curated/<Brand>_<YYYY-MM>.json` — hand-written call-highlight input (`listen`, two real call reasons) — heytruffle pipeline only, gitignored (real client data)

## CSV Requirements

The Metabase export must use this query pattern (update `period` for each month):

```sql
SELECT u."name" AS USER_NAME, l.location_name AS LOCATION_NAME, m.*
FROM metric m
JOIN location l ON l.id = m."locationId"
JOIN "user" u ON u.id = l."userId"
WHERE m."period" = '2026-02'
```

Brands with fewer than 50 total calls (`MIN_CALLS = 50` in `parse_metrics_csv.py`) are skipped automatically.

---
> Source: [twais-resto/reports-generation](https://github.com/twais-resto/reports-generation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
