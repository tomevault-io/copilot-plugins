## ceie-proceedings-migration

> Rules for file organization and temporary files management


# File Organization Rules

## Temporary Files Management

### Rule: All temporary files MUST be created in temp/ directory

All temporary files created during development, debugging, or testing MUST be placed in the `temp/` directory:

**Temporary files include:**

- Temporary Python scripts (e.g., `debug_issue.py`, `test_feature.py`)
- Temporary documentation (e.g., `ANALYSIS_RESULTS.md`, `DEBUG_NOTES.md`)
- Debug logs (e.g., `debug_output.log`)
- Temporary CSV files for analysis (e.g., `analysis_results.csv`)
- Any other temporary artifacts

**Correct locations:**

```
✅ temp/debug_cnpj_issue.py
✅ temp/ANALYSIS_RESULTS_20251021.md
✅ temp/test_output.log
✅ temp/orphan_companies_analysis.csv
```

**Incorrect locations:**

```
❌ debug_cnpj_issue.py                          (project root)
❌ ANALYSIS_RESULTS_20251021.md                 (project root)
❌ collection/scripts/maintenance/debug_*.py    (maintenance folder)
❌ collection/scripts/test_*.py                 (scripts folder)
```

### Exception: Permanent Files

Only create files outside `temp/` if they are:

- Part of the permanent codebase (agents, modules, libraries)
- Official CLI tools in `collection/cli/`
- Essential maintenance scripts in `collection/scripts/maintenance/`
- Documentation that should be version-controlled

---

## PDF File Naming

### Rule: All PDFs must follow XXXX_YYYY.pdf format

When downloading or creating PDF files for ESG reports:

**Format:** `XXXX_YYYY.pdf`

- `XXXX`: 4-letter company code (e.g., PETR, VALE, BBDC)
- `YYYY`: 4-digit year (e.g., 2023, 2024, 2025)

**Location Flow:**

1. **New downloads** → `data/reports/found/` (Status: FOUND)
2. **After validation** → `data/reports/confirmed/` (Status: CONFIRMED)
3. **After processing** → `data/reports/processed/XXXX_YYYY.md` (Status: COMPLETED)

**Examples:**

```
✅ data/reports/found/PETR_2023.pdf      (new download)
✅ data/reports/confirmed/VALE_2024.pdf  (validated)
✅ data/reports/processed/BBDC_2022.md   (processed)
```

**Incorrect formats:**

```
❌ data/reports/found/cvm/2023/PETR/report.pdf  (nested structure)
❌ data/reports/found/20251021_PETR_2023.pdf    (timestamp prefix)
❌ data/reports/found/PETR_2023_cvm_report.pdf  (extra suffixes)
```

---

## CSV File Standards

### Rule: Use semicolon delimiter and avoid commas in field values

**CSV Structure:**

- **Delimiter:** `;` (semicolon)
- **Encoding:** UTF-8
- **Field values:** Must NOT contain commas (breaks CSV structure)

**Key CSV files:**

- `storage/structured/companies.csv` - Company master data
- `storage/structured/report_status.csv` - Report tracking (14 columns)

**report_status.csv structure:**

- **Unique ID:** `Company_Code` + `Year` (no separate Entity_ID column)
- **Required fields:** 14 columns
- All lines must have consistent field count

---

## Script Organization

### Rule: Only essential, reusable scripts in collection/scripts/maintenance/

**Essential scripts (currently maintained):**

- `audit_data_integrity.py` - Periodic integrity checks

**Where to put scripts:**

- **Temporary/one-time scripts** → `temp/`
- **Experimental features** → `collection/scripts/experimental_agents/`
- **Official CLIs** → `collection/cli/`
- **Essential maintenance** → `collection/scripts/maintenance/` (rarely!)

**Before adding to maintenance/:**
Ask yourself:

1. Will this script be used multiple times?
2. Is this a core system function?
3. Can't this be done via CLI or existing tools?

If answers are No/No/Yes → Put in `temp/` instead

---

## Data Integrity

### Rule: Maintain data consistency at all times

**When modifying CSV files:**

1. ✅ Always create backup in `storage/structured/backups/`
2. ✅ Ensure consistent field count (14 for report_status.csv)
3. ✅ Verify no duplicates (Company_Code + Year must be unique)
4. ✅ Use `;` delimiter, avoid `,` in field values

**When downloading PDFs:**

1. ✅ Use format `XXXX_YYYY.pdf`
2. ✅ Save to `data/reports/found/` (flat structure)
3. ✅ Register in `report_status.csv`
4. ✅ Verify file exists before adding to CSV

**When modifying companies.csv:**

1. ✅ Ensure unique Code and id fields
2. ✅ Use 2-letter ISO country codes
3. ✅ Use 4-letter alphabetic company codes
4. ✅ Maintain Status column (active/merged/extinct)

---

## Example Violations to Avoid

```python
# ❌ BAD: Creating temporary script in wrong location
# File: collection/scripts/maintenance/debug_cnpj_issue.py
def debug_cnpj():
    ...

# ✅ GOOD: Create in temp/
# File: temp/debug_cnpj_issue.py
def debug_cnpj():
    ...
```

```python
# ❌ BAD: Downloading with wrong naming
pdf_path = "data/reports/found/cvm/2023/PETR/timestamp_report.pdf"

# ✅ GOOD: Standard naming and location
pdf_path = "data/reports/found/PETR_2023.pdf"
```

```python
# ❌ BAD: CSV field with comma
notes = "Downloaded from CVM, validated successfully"  # Breaks CSV!

# ✅ GOOD: No commas in field values
notes = "Downloaded from CVM | validated successfully"
```

---

**Last updated:** 2025-10-21

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/patriciajaques) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
