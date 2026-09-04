## duckdb-data-diff

> CLAUDE.md: DUCKDB DATA COMPARISON - ARCHITECTURAL STANDARD

CLAUDE.md: DUCKDB DATA COMPARISON - ARCHITECTURAL STANDARD
IDENTITY: You are the Lead DuckDB Configuration Architect for this project. Your primary focus is on Reliability > Features and enforcing the SOLID/DRY principles.

IMPORTANT: ALWAYS REFERENCE THIS FILE. YOU MUST follow all instructions below.

I. CORE PHILOSOPHY (The Non-Negotiables)
Think like a Data Engineer + Software Architect:

Single Responsibility: Each component is testable in isolation (SOLID).

DRY Principle: Do not repeat logic.

Fail Fast: Use clear, actionable error messages and never fail silently.

Idempotency: Running the same operation twice must yield the same result.

Reproducibility: Results must be verifiable every time.

II. CRITICAL CONSTRAINTS (Priority 1)
These constraints are non-negotiable and prioritize memory/performance.

File/Function Size:

MAX 300 lines per file.

MAX 50 lines per function/method.

Memory-Efficient Processing: All data handling logic MUST be designed for large files. Use chunked processing for tables > 100K rows.

Data Profiling Constraint: When profiling raw data files for column mapping (e.g., using pandas read_csv/excel), YOU MUST read a minimum sample size of 5,000 rows (nrows=5000) to ensure accurate column detection and avoid data type errors from sparse columns.

# KEY LEARNING: When matching inventory/financial data, always prioritize the stable, numeric 'System ID' (e.g., serial_number_id: 982) over the alphanumeric 'Business ID' (e.g., Serial/Lot Number: A240307704) to prevent fundamental data incompatibility errors.

SQL Generation Consistency: ALL SQL generation in comparator.py (including chunked methods) MUST use the _get_right_column() helper for ALL JOIN and WHERE conditions to correctly apply column mapping.

III. CODING STANDARDS & QUALITY
Requirement	Standard
Structure	Use Type Hints and Google-style docstrings for all functions/classes.
Testing	MANDATORY TDD. All new features/fixes MUST follow: Write Tests → Commit → Code → Iterate → Commit.
Utility Functions	Utility functions with a single responsibility (e.g., _sanitize_table_name, _get_file_size) MUST be defined using a leading underscore for internal use and must handle failure modes explicitly (e.g., gracefully handling empty input or path errors).
Progress	Use rich_progress.py for user feedback (progress bars, spinners).

IV. ERROR HANDLING PATTERN
All errors MUST adhere to a specific format that provides actionable context:

[ERROR TYPE] Clear description. Suggestion: How to fix it.
V. ESSENTIAL COMMANDS
Command	Purpose
/status	Check current findings and session state.
/clear	MANDATORY between major tasks (e.g., after plan approval, before TDD for a new component).
/help	Display available commands and examples.

VI. SESSION SUMMARY AND CURRENT STATUS (Updated: 2025-09-25)

RESOLVED ISSUES:
1. Binder Error Fix: Column names are now properly normalized (snake_case) during SQL generation using normalize_column_name() helper
2. Missing Value Differences Report: Added comprehensive value_differences.csv export with professional headers
3. Column Normalization Duplicates: Fixed by properly dropping tables/views before renaming in stager.py
4. Date Type Conversion Error: All columns now CAST to VARCHAR before comparison to avoid type mismatches
5. Boolean Value Handling: Normalized boolean representations (true/t/false/f) before comparison
6. Column Mapping Implementation: Fixed _determine_value_columns to use column mapping via _get_right_column() helper
7. Key Detection with Mapping: Updated _determine_keys to consider mapped columns when auto-detecting keys
8. **MAJOR ARCHITECTURAL FIX (2025-09-25)**: Column Mapping Normalization and Key Priority Resolution
   - **Primary Issue**: Only 2 columns compared despite 15+ approved mappings (configuration-runtime mismatch)
   - **Secondary Issue**: SQL joins failed with "Table 'r' does not have column 'from'" despite valid column mappings
   - **Root Causes**: (1) Original column names in mappings vs normalized names in staged tables, (2) Key column conflict resolution removed primary key mapping when multiple columns map to same right column
   - **Solutions Applied**: (1) Normalization Consistency Pattern in menu.py:_create_interactive_config(), (2) Key Column Priority Pattern in conflict resolution
   - **Impact**: Full column mapping functionality restored - 13+ mapped columns now successfully compared instead of only 2 exact matches
   - **Verification**: End-to-end interactive verification confirmed both fixes working, SQL generates correct "l.from = r.author" joins
   - **Tests**: 10/10 comprehensive TDD tests pass (5 normalization + 5 key priority tests)

NEW ARCHITECTURAL CONVENTIONS:
1. Report Headers: Use human-readable headers (e.g., "Left is_incoming" instead of "left_is_incoming")
2. Encoding Handling: Try multiple encodings (utf-8, utf-8-sig, latin-1, iso-8859-1, cp1252) in file_reader.py
3. Text Normalization: Created text_normalizer.py utility for Unicode normalization and character replacement
4. Column Mapping Flow: Right dataset's column_map maps right columns to left columns (e.g., 'author' -> 'from')
5. Staged Column Names: All column names are normalized to snake_case during staging process
6. SQL Safety: Always quote column names with special characters/spaces using quote_identifier()
7. Chunked Processing: Automatic chunking for datasets > 25,000 rows with progress indicators
8. **NORMALIZATION CONSISTENCY PATTERN**: When building dataset configurations from interactive matches, always normalize both sides of the column map and the dataset key_columns array to ensure consistency with staged data. This prevents column mapping lookup failures in comparator.py.
9. **KEY COLUMN PRIORITY PATTERN**: In interactive column mapping conflict resolution, any key column defined in the validated_keys list receives absolute priority and always replaces any non-key conflict, regardless of confidence score. This ensures SQL joins succeed by preserving key column mappings.
10. **STAGED KEY CONSISTENCY PATTERN**: In key validation, user-selected key column names must be transformed to match actual staged table column names. For left tables (no column_map), normalize key columns using `normalize_column_name()`. For right tables (with column_map), apply column mapping first then normalize the result. This prevents key validation failures caused by column name mismatches between user selection and staged data.
11. **INTERACTIVE MENU KEY NORMALIZATION PATTERN**: Interactive menu interfaces must present normalized column names that match staged table schema to users for key selection. Build a mapping from normalized names to original names for display purposes, present the normalized names as selectable options, and return original names for backward compatibility. This prevents "Column not found" errors when users select keys that exist in the menu but not in staged tables.
12. **EMPTY MATCHES SAFETY PATTERN**: Interactive menu methods must validate that reviewed_matches is not empty before proceeding with key selection. If empty, return gracefully with clear error message rather than entering infinite loops. This prevents complete system failures when no column matches are approved and ensures reliable operation across all dataset scenarios.
13. **SCHEMA FINGERPRINT VALIDATION PATTERN**: DataStager must detect schema drift and file changes by comparing current source file metadata (columns, modification time) against stored metadata (.meta files). Implementation includes _read_source_columns() to extract schema, _should_restage() to detect changes, and _write_metadata() to persist fingerprints. This prevents stale cache reuse that causes Binder Errors and ensures staged data reflects current source file schema.
14. **FAIL FAST COMPARISON PATTERN**: DataComparator must immediately halt pipeline execution with KeyValidationError when key validation fails (is_valid=False), preventing downstream processing on invalid data. Error messages must follow mandatory format: "[KEY VALIDATION ERROR] Clear description. Suggestion: How to fix it." This ensures data integrity and provides actionable feedback for resolution.
15. **NORMALIZED INVERSE MAPPING PATTERN**: In KeyValidator._get_staged_key_columns for right tables, create a fully normalized column mapping where both keys (right columns) and values (left columns) are normalized before performing inverse lookups. Normalize user input key_columns before lookup to ensure correct right table column discovery. This prevents SQL generation failures where wrong column names are used in JOIN conditions due to case/spacing mismatches between user input and stored mappings.
16. **REPORT FIDELITY PATTERN**: DataComparator.export_differences supports enhanced reporting with configurable export formats and detailed annotations. Implementation includes (1) Chunked Full Exports via _export_full_csv() helper with configurable chunk_export_size for large datasets, (2) Schema-Annotated Previews with CTE-based SQL generating "Entire Column Different" flags to identify columns where all values differ, (3) Smart Preview Logic using UNION ALL to combine summaries (entire_column=true), samples (entire_column=false), and partials with deterministic ordering, and (4) Configuration-driven behavior via ComparisonConfig attributes (csv_preview_limit, export_full, annotate_entire_column, enable_smart_preview). This eliminates confusion from preview limits and provides complete data exports when needed while maintaining performance for large datasets.
17. **SQL QUERY SANITIZATION PATTERN**: DataComparator._export_full_csv implements mandatory SQL query sanitization to prevent parser errors during chunked export operations. Implementation includes (1) _strip_trailing_semicolon() helper function that removes trailing semicolons and whitespace from base queries, (2) Query wrapping in subselects "SELECT * FROM (clean_query) q" before applying ORDER/LIMIT/OFFSET clauses, and (3) Consistent application of sanitization to both count queries and chunked export queries. This prevents "Parser Error: syntax error at or near 'ORDER'" regressions that occur when base queries contain trailing semicolons and are incorrectly wrapped with chunking clauses. The pattern ensures safe SQL composition for large dataset exports while maintaining backward compatibility.

18. **HYBRID FULL EXPORT PATTERN**: DataComparator.export_differences implements **HYBRID FULL EXPORT** to ensure full exports are never empty when preview has data. Implementation includes (1) **Hybrid Logic**: Full exports (`value_differences_full.csv`) combine collapsed summaries (one row per entirely-different column) with ALL partial differences (no preview cap), using UNION ALL structure, (2) **Permanent Preview Collapse**: Preview exports (`value_differences.csv`) use permanent collapse showing one summary row per entirely-different column plus capped samples of partial differences, (3) **Never Empty Promise**: Full exports always contain data when preview shows differences, solving the critical bug where full exports were empty with only partial differences, (4) **Optional Audit Export**: `export_rowlevel_audit_full=True` generates separate `value_differences_full_audit_partNNN.csv` files containing complete row-level detail for users who need full data, (5) **Deprecated Configuration**: Both `collapse_entire_column_in_preview` and `collapse_entire_column_in_full` flags have been **REMOVED** - behavior is now permanent and no configuration is needed, (6) **Professional Architecture**: Maintains all SQL safety helpers (qident(), qpath(), _strip_trailing_semicolon()), QUALIFY fallback, chunked exports, and deterministic ordering with sample discriminators for hybrid queries. This pattern ensures users get **actionable** collapsed summaries for entirely-different columns plus **complete** data for partial differences.

CURRENT STATE:
- ✅ **ARCHITECTURE FULLY RESTORED**: Both critical column mapping bugs resolved (2025-09-25)
- ✅ **HYBRID FULL EXPORT IMPLEMENTED**: Full exports never empty when preview has data (2025-09-27)
- Column mapping functionality working at full capacity with 13+ mapped columns
- Interactive menu applies normalization consistency and key column priority patterns
- Configuration-runtime integration verified through comprehensive TDD test suites
- SQL generation robust with proper join conditions for mapped key columns
- Debug logging enhanced for full traceability of mapping transformations and conflict resolution
- **NEW**: `value_differences.csv` (preview) shows collapsed summaries + capped partial samples
- **NEW**: `value_differences_full.csv` (full) shows collapsed summaries + ALL partial differences (hybrid approach)
- **NEW**: `export_rowlevel_audit_full=True` provides opt-in complete row-level detail via `value_differences_full_audit_partNNN.csv`
- **FIXED**: Critical bug where full exports were empty when only partial differences existed
- **DEPRECATED**: Both `collapse_entire_column_in_preview` and `collapse_entire_column_in_full` flags removed - behavior is permanent

VII. ACTIVE PLAN / TASKS

## FULL DATASET VALIDATION PLAN (Data Quality Assurance Phase)
*Status: IN EXECUTION | Phase: Environment Setup*

### 🎯 **1. DATASET PRIORITIZATION STRATEGY**
**Method: Risk-Based Incremental Validation**

**Priority Queue Logic:**
```
Priority = (File_Size_KB + Column_Count * 10 + Complexity_Score * 5)
```

**Sorting Criteria:**
1. **Tier 1 (Smallest First)**: < 1MB, < 10 columns, no column mapping
2. **Tier 2 (Medium)**: 1-10MB, 10-25 columns, simple column mapping  
3. **Tier 3 (Large)**: 10-100MB, 25+ columns, complex mappings
4. **Tier 4 (Critical)**: > 100MB, hierarchical data, composite keys

### 📊 **2. TRACKING CHECKLIST SYSTEM**
**Artifact: `VALIDATION_PROGRESS.md` (Separate Working File)**

### 🚨 **3. ERROR HANDLING PROTOCOL**
**Mandatory CLAUDE.md Error Format:**
```
[ERROR_TYPE] Clear description. Suggestion: How to fix it.
```

**Categories:**
- **[CONFIGURATION ERROR]**: Dataset config malformed
- **[DATA FORMAT ERROR]**: File encoding/structure issue  
- **[COLUMN MAPPING ERROR]**: Mapping validation failure
- **[PERFORMANCE ERROR]**: Memory/timeout exceeded
- **[SQL GENERATION ERROR]**: Query construction failure

### ✅ **4. COMPLETION CRITERIA**
**Phase Complete When:**
- **Primary Goal**: 20/20 datasets successfully compared
- **Quality Gate**: All error types documented and resolved
- **Performance Threshold**: No dataset exceeds 10-minute processing
- **Memory Constraint**: No comparison exceeds 8GB peak memory

---

# CRITICAL STABILITY NOTE: Hierarchy normalization has failed persistence repeatedly (likely an environment/staging bypass issue). Code is confirmed correct. Full stability validation is now the ONLY priority.

## PREVIOUS TASKS (Completed)
1. **TDD: Composite Key Selection**: Write tests and implement logic for selecting multiple key columns via the interactive menu (`menu.py:select_composite_key_interactively`) and test the validation of composite key uniqueness (`key_validator.py:_validate_composite_key`).

# ARCHITECTURE SYSTEM FIX: Enforced normalization of config column names ('if col == normalize_column_name(column):') in stager.py to guarantee normalizer application. This resolved the persistent hierarchy normalization failure.
- to memorize
- to memorize "How we handled permanent collapse logic for reports"
- to memorize "# CRITICAL BUG: Investigate intermittent false-negatives in "only in" reports and false-positives in "value differences" reports. Hypothesis: The `comparator.py` robust comparison logic or the key derivation process in `key_validator.py` is failing to correctly handle string/number type coercion and normalization across joins, particularly when involving configured column mappings."
- to memorize # CRITICAL BUG: **ROW MISMATCH ISSUE (ACTIVE).** Comparison reports are literally comparing the wrong rows (false difference reports) or marking matching rows as "only in" (false negatives). This suggests a severe logic error in the primary key join or the subsequent value comparison process.

---
> Source: [JoelAbbott/duckdb-data-diff](https://github.com/JoelAbbott/duckdb-data-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
