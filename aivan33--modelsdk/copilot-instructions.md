## modelsdk

> Parser module conventions (formula extraction, templates, dependencies, groups)


# Parser Rules

## Formula Template Extraction
- Parse cell references with regex. Handle both individual refs (Sheet!ColRow) and range refs (Sheet!Col1Row1:Col2Row2).
- Absolute refs ($-prefixed columns): keep as-is in template.
- Relative refs: compute offset from formula's own column. Replace with {col+offset} or {col} if offset=0.
- Range references are a single unit: =SUM(G674:G677) → =SUM({col}674:{col}677). Not two separate refs.
- Template consistency check: extract templates for col N and col N+1. If they differ, record a consistency_break.

## Logical Groups
- A logical group = all rows referencing the same (source_sheet, source_row).
- Computed from the dependency graph, NOT from label formatting or indentation.
- Label hierarchy (bold parents, indentation) is metadata stored in parent_label. Not used for grouping.
- A row can belong to multiple logical groups.

## Scenario Selector Detection
- Don't hardcode $D$2 or any specific cell.
- Scan all formulas for IF() conditions. Extract absolute cell refs used as the condition test.
- Any absolute ref appearing in IF conditions across 10+ rows is a scenario selector.
- Store in model metadata: scenario_selectors list.

## Dual Load Pattern (openpyxl)
- data_only=False → formulas (for structural parser)
- data_only=True → cached values (for comparison module)
- Always use read_only=True for both loads.
- data_only=True returns None if file wasn't saved by Excel. Handle gracefully.

## Incremental Parsing
- parse_incremental(source, cached_structure) diffs formula strings against cache.
- If formula at (sheet, row, col) matches cache → skip row entirely.
- Changed/new rows get full-parsed. Deleted rows removed from structure.

## Formula Normalization (for Google Sheets, Phase 6)
- normalize_formula(formula_string, source_format) runs BEFORE template extraction.
- Handles: semicolon→comma separators, ARRAYFORMULA unwrapping, Sheets-specific functions.
- Financial model formulas (SUM, IF, SUMIF, VLOOKUP) are identical between Excel and Sheets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aivan33) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
