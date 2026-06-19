## excel-workbook-editing

> Design, edit, debug, reconcile, and validate Excel workbooks, workbook-generation scripts, and connected Google Sheets. Use when the task involves .xlsx files, Excel formulas, sheet layouts, named ranges, lookup logic, cross-sheet references, workbook templates, carryover flows, Google Sheets ranges, live spreadsheet formulas, validations, protected ranges, IMPORTRANGE, Apps Script-connected sheets, large or timeout-prone Sheets, external loading states, rollback snapshots, or spreadsheet review packages. Avoid for CSV/TSV-only tasks without spreadsheet-specific formulas, layout, validation, or connected-document concerns.


# Spreadsheet Workbook Editing

Treat spreadsheets as calculation and review systems, not just tabular files.
For existing artifacts, preserve both the intended logic and the artifact identity:

- Excel workbooks are `.xlsx` files whose formula behavior must be validated by Excel.
- Existing Google Sheets are live, connected documents whose `spreadsheetId`, `sheetId` values, permissions, formulas, protections, Apps Script bindings, and external dependencies must not be broken by replacement-style workflows.

## Quick Start

1. Identify the artifact type: Excel workbook, workbook generator, existing Google Sheet, new standalone Google Sheet, or review package.
2. Separate raw inputs, parameters, intermediate calculations, derived outputs, and any prior-period carryovers.
3. Read [references/spreadsheet-principles.md](references/spreadsheet-principles.md) for shared spreadsheet rules.
4. Route to the right artifact-specific workflow:
   - Excel workbook or `.xlsx` generator: read [references/excel-workbook-principles.md](references/excel-workbook-principles.md).
   - Existing Google Sheet: read [references/connected-google-sheets-principles.md](references/connected-google-sheets-principles.md).
   - Review package or CLI preview: read [references/spreadsheet-review-package.md](references/spreadsheet-review-package.md).
5. Choose the lowest-risk edit surface: workbook formulas, named ranges, template layout, generator script, Google Sheets range patch, or validation automation.

## Task Routing

### Excel Workbooks

- **READ**: Before editing an unfamiliar workbook, run `scripts/inspect_workbook.py` to summarize sheets, dimensions, formulas, named ranges, merged ranges, tables, validations, and hidden structure.
- **EDIT**: Make deterministic workbook or generator changes in code, usually with `openpyxl` or the repo's workbook-generation layer.
- **VALIDATE**: Recalculate formula results with the real Microsoft Excel engine, preferably through `scripts/excel_engine_sample.py` for unattended temporary-copy validation.
- **SCAN**: After recalculation, run `scripts/formula_error_scan.py` to check formula and literal error cells such as `#REF!`, `#VALUE!`, `#N/A`, and `#NAME?`.
- **RECONCILE**: For approved-workbook comparisons, use a three-surface comparison: approved or golden workbook, authoritative code or calculation script, and newly generated workbook.

### Connected Google Sheets

- **GROUND**: Confirm the exact spreadsheet URL or id, tab names, `sheetId` values, target ranges, headers, formulas, validations, protected ranges, and named ranges before editing.
- **PRESERVE**: Do not download an existing Google Sheet to `.xlsx`, edit it locally, and upload it back as the default workflow. Existing Sheets are identity-bearing connected documents.
- **RISK SCAN**: Look for `IMPORTRANGE`, import functions, `QUERY`, `ARRAYFORMULA`, `INDIRECT`, custom functions, Apps Script signals, protected ranges, validation-backed cells, and external dashboards or automations before planning writes.
- **TIMEBOX**: For large or externally linked Sheets, plan reads, writes, retries, and post-write polling under explicit timeout and quota budgets.
- **PATCH**: Prefer narrow Google Sheets API or connector edits against existing ranges. Avoid deleting and recreating tabs, replacing whole sheets, or overwriting formulas with displayed values unless the user explicitly requests that behavior.
- **READBACK**: Re-read the changed cells, formulas, validations, import/load states, and key dependent outputs from the live Google Sheet after writing.

### Review Packages

- **PACKAGE**: When the user needs CLI-visible evidence, produce a self-contained review package with HTML previews, JSON structure, formula/dependency risk logs, key values, and before/after summaries.
- **SEPARATE**: Treat visualization as evidence, not calculation proof. Report the calculation engine or live readback used for validation.

## Editing Defaults

- Preserve a traceable path from inputs to outputs.
- Keep important parameters visible and centralized.
- When modifying an existing template, study and preserve its established formatting, sheet conventions, named-range patterns, and reviewer-facing layout unless the user explicitly asks to change them.
- Do not add a second workbook-only logic path that silently diverges from the authoritative logic.
- Do not hide known input limitations inside ad hoc override formulas.
- Prefer compatibility-safe formulas over newer functions when the target spreadsheet environment is uncertain.
- For recurring period workbooks, separate prior-period carry-ins, current-period raw inputs, and next-period carry-outs explicitly instead of mixing them into one surface.
- If the workbook is meant for auditability, keep the workbook as an explanation surface for the logic rather than a thin shell around copied totals.
- When reconciliation against an approved workbook is required, prefer an explicit `raw -> limitation -> adjusted` presentation rather than silently baking manual patches into formulas.
- For existing Google Sheets, preserve spreadsheet identity and connected behavior before optimizing for local convenience.

## Structural Patterns

For small workbooks, a simple `input -> calc -> output` layout is often enough.
For complex workbooks, separate these roles explicitly:

- input sheets
- config or parameter sheets
- intermediate calculation sheets
- bridge sheets for logic that would otherwise become opaque
- output or export sheets
- known limitation sheets when some cases are intentionally excluded or manually handled

Use visible intermediate sheets when a single formula chain becomes hard to audit or debug.
Keep prior-period detail visible when carryover or opening-balance logic affects current results.
When operational raw inputs are messy or inconsistent across sources, add a standard input sheet before the bridge layer instead of wiring formulas directly to the raw extract.
If a bridge sheet can legitimately have zero source rows in some periods, design it so that downstream formulas still evaluate to explicit zeroes rather than empty or missing objects.

## Formula Defaults

- Use defined names for lookup ranges, aggregation ranges, config cells, and single-value carryovers.
- Avoid hard-coded column letters and parameter cells such as `A:A`, `BG:BG`, or `Config!B4`.
- Normalize lookup IDs with explicit text key columns such as `transaction_id_key`.
- Prefer `INDEX/MATCH + IFERROR` over `XLOOKUP` for workbook compatibility.
- Prefer `SUMPRODUCT` over `SUMIFS` when aggregating named ranges.
- Keep config dates as real Excel date cells, not strings.
- Avoid heavy whole-column diagnostics unless you have confirmed the recalculation cost is acceptable.
- Normalize date comparisons when time fractions may exist in Excel datetime values; a displayed date is not always a date-only value for formula purposes.
- When matching duplicate keys, verify whether Excel is using first-hit behavior and keep any Python-side mirrors aligned to the same selection rule.
- For aggregates fed by bridge sheets that may be empty, prefer explicit coercion patterns such as `IFERROR`, `N()`, or `0+named_range` so outputs close to numeric zero instead of blank or missing values.
- Treat source-file selection as part of workbook correctness: a broken source resolver can look exactly like a workbook formula defect.
- In Google Sheets, treat `IMPORTRANGE`, import functions, custom functions, array formulas, and protected/validated cells as live dependencies that must be preserved or explicitly changed.

## Tool Selection

Choose tools by task type rather than by habit.

- Use `openpyxl` or a workbook-generation script for deterministic structural edits such as adding sheets, writing formulas, creating named ranges, filling tables, and producing repeatable workbook outputs.
- Use `scripts/inspect_workbook.py` for structure discovery before changing a non-trivial or unfamiliar workbook.
- Use `scripts/formula_error_scan.py` after recalculation to catch workbook-wide formula and literal error cells.
- Use desktop Excel automation only as a control layer for the real Excel application when you need Excel to open, recalculate, save, or expose a narrow set of computed cell values.
- For unattended local validation, prefer the Python wrapper `scripts/excel_engine_sample.py` before calling OS-specific helpers directly. It opens a temporary workbook copy, runs the real Excel engine, samples narrow cells, and removes the copy.
- Use the Excel application when you need authoritative recalculation, visual inspection, feature behavior that only Excel can express, or confirmation that a human reviewer can actually follow the workbook.
- Keep bulk data transformation and workbook construction out of desktop automation and Excel UI when code can do it more safely and repeatably.
- Use the bundled desktop helpers only when you need a read-only recalc-and-sample loop in real Excel: `scripts/excel_recalculate_and_sample.applescript` on macOS, or `scripts/excel_recalculate_and_sample.ps1` on Windows.
- Use Google Sheets connectors or APIs for existing Google Sheets edits so the live document identity, range metadata, and dependency graph are preserved.
- Use `.xlsx` to Google Sheets import only for new standalone Sheets or explicit replacement/clone workflows, not for ordinary edits to connected existing spreadsheets.

## Validation Workflow

1. Confirm the intended logic from the authoritative source and the artifact identity that must be preserved.
2. Inspect workbook structure before editing when the workbook is unfamiliar or template-like.
3. Make the workbook or script change.
4. Recalculate with the real Excel engine before trusting results.
5. Inspect representative rows, key aggregate cells, likely edge cases, and workbook-wide formula errors.
6. Compare results against the authoritative source.
7. Fix formulas, names, sheet wiring, or automation and repeat.

For existing Google Sheets, use live readback instead of local workbook validation:

1. Read metadata and target cells from the existing spreadsheet.
2. Inspect dependency, validation, and protection risks for the target area.
3. Apply the smallest coherent range update.
4. Poll external-data and custom-function outputs only within a bounded wait plan.
5. Re-read changed cells and dependent outputs from the live spreadsheet.
6. Confirm `spreadsheetId`, `sheetId` values, formulas, validations, and protected ranges were preserved unless deliberately changed.

For recurring reconciliations, prefer a three-surface comparison when possible:

- approved or golden workbook
- authoritative code or calculation script
- newly generated workbook

This separates logic bugs, source gaps, workbook wiring bugs, and Excel behavior differences faster than comparing only one pair of outputs.

## Automation Cautions

- `openpyxl` and similar libraries can write formulas but do not prove calculated results.
- `.xlsx` round-trips do not preserve existing Google Sheets identity or every connected behavior.
- Assume Excel automation is fragile while the target workbook is being edited by a user.
- Avoid validation loops that depend on active sheet focus or manual save timing.
- Read only the cells you need during automated checks; broad workbook scans are slow and noisy.
- Distinguish first-run desktop automation permission from file-access prompts. The wrapper can reduce repeated file-access prompts or source workbook locks, but a new machine may still need a one-time permission grant for the terminal or agent host to control Microsoft Excel.
- If Excel file access prompts repeat for project paths, validate a temporary copy instead of opening the source workbook directly.
- If the real Excel application is unavailable, treat formula-result validation as incomplete and say so explicitly.
- If Google Sheets Apps Script bindings, installable triggers, add-ons, or webhooks may matter but cannot be inspected, mark that as an unverified connected-document risk.
- If a Google Sheets API request times out, returns `429`, or returns `503`, reduce request size or spreadsheet complexity before repeating the same operation.

## Execution Examples

### Example 1: Add a new input column and wire it into formulas

1. Update the generator script or use `openpyxl` to add the column, related formulas, and named ranges.
2. Normalize any lookup key needed for joins.
3. Open the workbook in Excel and recalculate.
4. Check a few representative rows plus one aggregate cell.

Primary tools:

- `openpyxl` or generator script for the edit
- Excel for recalculation and review

### Example 2: Debug a workbook that shows unexpected `#N/A`

1. Identify which lookup or aggregation path produces the error.
2. Inspect whether the key type, named range, or date type is inconsistent.
3. Recalculate in Excel.
4. Read only the affected cells and a few upstream cells.
5. Replace fragile formulas such as `XLOOKUP` or hard-coded references if needed.

Primary tools:

- Excel for authoritative results
- desktop Excel automation for repeatable recalc-and-read loops on macOS or Windows
- `openpyxl` only for applying the eventual structural fix

Example command:

```bash
python3 /path/to/excel-workbook-editing/scripts/excel_engine_sample.py \
  /path/to/workbook.xlsx \
  1 \
  A1 \
  B2 \
  C10
```

### Example 3: Generate a monthly workbook from source data

1. Build the workbook in code.
2. Write formulas, names, and sheet structure with `openpyxl` or another generator layer.
3. Open the output in Excel.
4. Recalculate and compare key outputs against the source of truth.

Primary tools:

- generator script plus `openpyxl` for creation
- Excel for final validation

### Example 4: Check whether a workbook is visually and logically reviewable

1. Open the workbook in Excel.
2. Inspect whether inputs, parameters, intermediate logic, and outputs are visually separated.
3. Confirm that major totals can be traced back through visible intermediate steps.
4. Only after that, decide whether a structural refactor is needed in code.

Primary tools:

- Excel first
- code second if the workbook needs repeatable repair

### Example 5: Edit an existing Google Sheet with IMPORTRANGE dependencies

1. Confirm the spreadsheet URL, target tab, target range, and intended edit.
2. Read spreadsheet metadata, target cells, formulas, validations, protections, and named ranges.
3. Identify `IMPORTRANGE`, import functions, `ARRAYFORMULA`, Apps Script custom functions, and dependent outputs near the edit.
4. Classify import cells as loaded, loading, permission-blocked, source-blocked, or broken before writing around them.
5. Apply a narrow range-scoped update through the Google Sheets connector or API.
6. Re-read the changed cells and dependent outputs from the live spreadsheet under a bounded polling plan.

Primary tools:

- Google Sheets connector or API for in-place edits
- review package output for CLI-visible evidence when needed

## Common Failure Modes

- hard-coded references drifting after row or column insertions
- `XLOOKUP` or newer functions behaving inconsistently across environments
- named-range aggregations returning unexpected zeroes
- numeric-versus-text key mismatches breaking joins
- dates stored as text causing comparison errors
- copied formulas silently pointing at the wrong sheet or period
- empty bridge ranges causing downstream aggregates to collapse into blanks or missing values
- first-hit versus last-hit mismatches between Excel formulas and Python-side joins
- datetime fractions causing period cutoff comparisons to behave differently from displayed dates
- sheets that exist structurally but are not actually wired into the active calculation path
- existing Google Sheets replaced by uploaded `.xlsx`, breaking `spreadsheetId`, `sheetId`, permissions, Apps Script bindings, or dependency approvals
- displayed values written over formulas in Google Sheets
- validation-backed Google Sheets cells updated with values that are not allowed by the live rule
- large Google Sheets requests timing out, hitting quota, or returning `503` because the spreadsheet or request is too complex
- external-data formulas left in `Loading...`, permission-blocked `#REF!`, or stale imported states after an edit
- Apps Script custom functions exceeding runtime, quota, authorization, or concurrent execution limits

## Done Criteria

- Excel recalculation or Google Sheets live readback matches the authoritative logic for the artifact being edited.
- Critical cells do not contain unexplained `#N/A`, blank values, stale formulas, or other formula errors.
- For Excel, workbook-wide formula error scan is clean or every remaining error is intentionally documented.
- The workbook or connected spreadsheet remains understandable to a human reviewer.
- Any intentional limitations or manual steps are explicitly visible.
- Important sheets are not just present; they are actually wired into the active calculation path and produce inspectable outputs.
- Existing Google Sheets retain their document identity and connected behavior unless replacement or disconnection was explicitly requested.
- Google Sheets timeout, quota, external-loading, and rollback status are checked or explicitly reported when they are relevant.

## Reference

Load [references/spreadsheet-principles.md](references/spreadsheet-principles.md) for shared spreadsheet identity, CRUD, agent workflow, and verification rules.
Load [references/excel-workbook-principles.md](references/excel-workbook-principles.md) for `.xlsx` structure, formula safety, Excel-engine validation, reconciliation, and desktop automation guidance.
Load [references/connected-google-sheets-principles.md](references/connected-google-sheets-principles.md) before inspecting or editing existing Google Sheets, including large, external, Apps Script-connected, or rollback-sensitive Sheets.
Load [references/spreadsheet-review-package.md](references/spreadsheet-review-package.md) when the user needs CLI-visible evidence, HTML previews, or review bundles for Excel or Google Sheets work.
Load [docs/codex-goal-chrome-extension-sheets-bridge.md](docs/codex-goal-chrome-extension-sheets-bridge.md) first when implementing the Chrome Extension and Native Messaging bridge for connected Google Sheets inspection or Apply Plan workflows. Use [docs/chrome-extension-sheets-bridge-design.md](docs/chrome-extension-sheets-bridge-design.md) for architecture context and [docs/chrome-extension-sheets-bridge-work-plan.md](docs/chrome-extension-sheets-bridge-work-plan.md) for roadmap context.

---
> Source: [kangminlee-maker/excel-workbook-editing](https://github.com/kangminlee-maker/excel-workbook-editing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
