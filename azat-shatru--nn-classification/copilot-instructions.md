## nn-classification

> A 10-stage ML preprocessing + classification pipeline built in **Plotly Dash** (Python).

# NN Pipeline — Project Brief

## What this is
A 10-stage ML preprocessing + classification pipeline built in **Plotly Dash** (Python).
Input: raw survey data (CSV/Excel) + questionnaire (.docx/.pdf/.xlsx/.txt).
Output: trained PyTorch neural network for multiclass classification.

## Repo
- **GitHub**: https://github.com/azat-shatru/NN_classification
- **Active branch**: `dash-app`
- **Main branch**: `main`

## How to run
```bash
venv/Scripts/python.exe dash_app/app.py
# → http://127.0.0.1:8050
```

## Project structure
```
dash_app/
  app.py              # Dash entry point, routing, sidebar, layout
  server_store.py     # In-memory store for DataFrames/models (avoids browser serialisation)
  pages/
    var_mapping.py    # Pre-stage: upload data + QNR, interactive tile mapping
    stage0_audit.py   # Data audit, column editor, drop/type assignment
    stage1_missing.py # Missing value imputation
    stage2_outliers.py
    stage2b_viz.py    # Visualisation
    stage3_encoding.py
    stage4_scaling.py
    stage5_correlation.py
    stage6_rf.py      # Random Forest feature importance
    stage7_factor.py  # Factor analysis
    stage8_combtest.py
    stage9_nn.py      # PyTorch NN training
  utils/
    qnr_parser.py     # Parses docx/pdf/xlsx/txt questionnaire → list of question dicts
    col_mapper.py     # Groups dataset columns by prefix, suggests var types
    stage2c_charts.py   # Charts Portal (Stage 2.6) — interactive charts + PPT export
  utils/
    qnr_parser.py     # Parses docx/pdf/xlsx/txt questionnaire → list of question dicts
    col_mapper.py     # Groups dataset columns by prefix, suggests var types
    ppt_export.py     # Builds native OOXML PowerPoint charts from codebook tiles
  assets/
    style.css         # All custom CSS
    dragdrop.js       # Chip drag-and-drop for Variable Mapping
    slide_order_dnd.js  # Slide row drag-and-drop for Charts Portal export
update_log.py         # Run this after every session to log changes to NN_Design_Tracker.xlsx
NN_Design_Tracker.xlsx  # Session log (Sheet: "Session Log"), next session = 24
```

## Architecture decisions
- **server_store.py** holds all DataFrames in process memory — do NOT put large data in `dcc.Store` (browser JSON)
- **app-state** (`dcc.Store`, session) holds lightweight pipeline flags (booleans, column name lists)
- **vm-state** (`dcc.Store`, memory) holds Variable Mapping mutations (deleted cols, type overrides, reassignments)
- Sidebar collapse state in `dcc.Store(id="sidebar-collapsed", storage_type="session")`
- All pages use `suppress_callback_exceptions=True` (set in app.py)
- `debug=False` — keep it off to suppress Plotly cloud prompt

## Key conventions
- Every `html.A` inside `dcc.Upload` must be `html.Span` (html.A with no href causes page refresh)
- Pattern-matching callbacks use `ctx.triggered_id` to identify which component fired
- `allow_duplicate=True` required when multiple callbacks write to the same Output
- After every coding session: update `update_log.py` and run it → logs to NN_Design_Tracker.xlsx

## Variable Mapping page — key details
- Layout: accordion card per question; body is a 4-col table (Variable | Options | Type | Question)
- Options appear as draggable chips in col 2; JS drag-and-drop (assets/dragdrop.js) via document-level event delegation
- Auto-assignment logic: 1 col → all opts; multi q_type + suffix → indexed opt; grid → all opts per col; scale/numeric → no chips
- Unassigned options pool row shown at bottom of each table when options remain unmatched
- "Save changes" button → clientside_callback reads DOM chip positions → vm-drag-assignments store → handle_save server callback
- `vm-state` uses `storage_type="session"` so assignments persist across page navigation
- `qnr_parser._parse_docx` returns `list[dict]` directly (two-pass: collect elements → group into questions)
- Table path: all paragraphs before table → question text; table first-col only → options
- No-table path: all paragraphs except last → question text; last → single option
- `_parse_table_for_options`: skips merged rows (vMerge continuation, gridSpan>1, single-cell disclaimer), reads col 0 only, stores `table_col_headers` + `table_n_cols`
- Grid-prefix expansion in `_default_option_assignments`: if n_vars ≈ n_opts × (n_table_cols-1) ±10%, expands options as "col_header opt" assigned 1:1 to matched cols

## Variable Mapping — option assignment details (session 21)
- `merge_qnr_with_metadata(questions, meta, col_groups)` in col_mapper.py is the main enrichment step
- **Option set rule**: if QNR options and col-E labels overlap (≥40% word match on any pair) → use INTERSECTION, QNR text canonical; if no overlap → QNR only
- **Grid expansion**: fires when `n_matched_vars == n_options × n_col_headers`; headers from QNR `table_col_headers`, or inferred from variable name suffixes; assigns `"{col_header} {option}"` sequentially
- **Zero unassigned guarantee**: suffix positional fallback (_1→opts[0]) then overassign all opts as last resort
- **Col B (var type)**: read from "Variable Label Information" sheet, mapped to canonical type via `_col_b_to_var_type()`; shown as default in Type dropdown
- **Debug logging**: `logging.getLogger("col_mapper")` at DEBUG level; per-question source + per-column assignment trace
- Sheet name lookup is case-insensitive
- `_mapping_source` field on enriched questions: `intersection` | `qnr_only` | `col_e_only` | `grid_expansion` | `n/a`

## Current state (last updated 2026-05-07, session 23)

### Session 23 — Charts Portal: drag-and-drop card + slide reorder

#### Chart card reordering
- `assets/chart_card_dnd.js`: HTML5 drag-and-drop for `.cp2c-chart-card` elements inside `#cp2c-cards-container`; uses mid-point x detection to insert before/after target in flex grid
- Cards rendered as flat flex container (replacing nested `dbc.Row/Col`); DOM order drives visual layout; card width computed from `cols_per_row` slider
- Each card has `draggable="true"`, `data-code` attr, and a `⠿` drag handle in the header
- **Save Order** button (top-right, next to Apply): clientside callback reads DOM `.cp2c-chart-card` order → saves list of codes to `cp2c-card-order` session store
- `render_charts` sorts `visible` tiles by stored card order before rendering; new/unordered tiles appended at end
- Card height fix: container uses `align-items:stretch`; each card wrapper is `display:flex; flex-direction:column`; `dbc.Card` has `height:100%` — all cards in a row stretch to match the tallest, preventing size inconsistency after reorder

#### Slide reorder
- `assets/slide_order_dnd.js`: HTML5 drag-and-drop for `.slide-order-row` elements in the Export card
- Slide Order panel shows each slide group as a draggable row (drag handle + Slide # + chart code chips)
- `cp2c-slide-order` (session Store): persists confirmed slide order
- **Confirm Order** button: clientside callback reads DOM row order → saves to `cp2c-slide-order` store
- `build_pptx()` in `ppt_export.py` accepts `slide_order` list; iterates slides in confirmed order; phantom slide numbers silently skipped

#### Other
- `render_charts` callback: 3 outputs (`cp2c-charts-grid`, `cp2c-summary`, `cp2c-slide-rows`); states include both `cp2c-card-order` and `cp2c-slide-order`
- Tested with real data: 1,200 rows × 713 cols, 54 questions, 49 codebook tiles

### Session 22
- QNR parse accuracy — Strategy 3 cross-validation added to col_mapper.py
- `merge_qnr_with_metadata()` now cross-validates option count vs dataset column count; warns when QNR options < 70% of implied count
- Warning format: `'{CODE}: QNR has {n} option(s) but {m} dataset column(s) suggest ~{expected} — QNR parsing may be incomplete'`
- var_mapping.py: red `⚠ incomplete parse` dbc.Badge on accordion card header + inline alert in body for any question with parse warnings
- Survey 2.docx and Raw data 2.xlsx updated by user

### Session 21
- Var Mapping option assignment rewritten: intersection/QNR-only rule, grid expansion, zero-unassigned safety pass
- parse_excel_metadata reads col B (var_types) in addition to col E (var_labels); case-insensitive sheet lookup
- refresh_mapping calls merge_qnr_with_metadata when both QNR and Excel metadata loaded
- Type dropdown prefers col-B type over heuristic inference
- qnr_parser._parse_docx: proper merge detection, table-path/no-table-path logic, grid-prefix expansion
- Stage 0: duplicate ID fixed; pages re-render on app-state change
- Stages 1–9: inherited from original Streamlit rewrite, not yet re-tested end-to-end

## Open tasks
- S10B root cause — Strategy 3 will surface mismatch count; investigate actual docx structure (page-break split table vs vMerge skip on rows 10-20)
- Test full pipeline end-to-end with Raw data 2.xlsx + Survey 2.docx
- A7 value=8 (NEE) custom missing value handling (Stage 1)
- Re-zip and upload to Google Drive
- Stages 1-9 not yet re-tested end-to-end

---
> Source: [azat-shatru/NN_classification](https://github.com/azat-shatru/NN_classification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
