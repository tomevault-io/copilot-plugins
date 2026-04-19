## broadband

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a German broadband data harmonization project that cleans and combines historical broadband availability data from 2005-2021 for German municipalities (Gemeinden). It transforms three separate data sources ("Pakets") into a unified panel dataset standardized to 2021 municipal boundaries.

**Key documentation**: See `docs/data_processing_pipeline.md` for comprehensive technical details.

## Architecture

### Data Processing Pipeline

The pipeline consists of 7 sequential R scripts in `src/main_pipeline/`:

1. `01_inspect_excel_sheets.R` - Scan Excel files to identify municipality data sheets
2. `02_process_paket_1.R` - Parse Paket 1 Excel files (2005-2014, historical naming)
3. `03_process_paket_2.R` - Parse Paket 2 Excel files (2010-2020, mixed formats)
4. `04_process_paket_3.R` - Parse Paket 3 CSV files (2021+, modern naming)
5. `05_combine_datasets.R` - Merge all three Pakets with validation
6. `06_standardize_ags_to_2021.R` - Remap to 2021 boundaries using Destatis crosswalks
7. `07_create_treatment_variables.R` - Create final wide-format panel

Each script outputs to `output/` (intermediate `.rds` files, final `.csv`).

### Key Directories

```
src/main_pipeline/     - Sequential processing scripts (run in order)
src/auxiliary/         - Verification, exploratory, and debug scripts
data/Paket_1/          - Historical Excel data (2005-2014)
data/Paket_2/          - Historical Excel data (2010-2020)
data/Paket_3/          - CSV data (2021+)
data/muni_mergers/     - Destatis municipal crosswalk files
output/                - Intermediate .rds and final CSV outputs
```

## Running the Pipeline

Run scripts sequentially in R/RStudio:
```r
source(here::here("src/main_pipeline/01_inspect_excel_sheets.R"))
source(here::here("src/main_pipeline/02_process_paket_1.R"))
# ... continue through 07
```

Or run individual scripts for debugging. Each script is self-contained with helper functions defined at the top.

## Core Concepts

- **AGS**: 8-digit official municipality key (Amtlicher Gemeindeschlüssel)
- **Paket**: One of three data bundles with different formats/periods
- **AGS standardization**: All data remapped to 2021 boundaries using population-weighted apportionment
- **Methodological breaks**: 2010 (baseline definition change), 2015 (provider change - flagged with `method_change_2015` dummy)

## Key Helper Functions

### `parse_broadband_variable(variable_name)`
Central function for interpreting varied broadband column names. Handles:
- Modern: `"Technology >= 100 Mbit/s"`
- Historical verf codes: `"verf_100_30"` (group_code_speed)
- Historical tech: `"DSL_16"`
- Prefixed: `"priv_dsl_100"`
- Year suffixes: `"verf_300_50_2010"` → extracts year

Returns tibble with `technology_group`, `speed_mbps_gte`, `year_from_variable`.

### `find_ags_column_name(col_names)`
Auto-detects AGS columns by pattern matching: "ags", "gemeindeschluessel", "gemeindeschlüssel", "gem", "kennziffer".

### `extract_year_from_filename(filename)` / `extract_year_for_sheet(sheet_name, filename_year)`
Cascading year extraction: sheet name → filename → variable suffix.

## Data Filtering Rules

- Only "privat" (private households) data - excludes gewerbe and mobilfunk
- Removes rows with NA `speed_mbps_gte` or `value` outside [0, 100]
- Final mobile technology filter as safeguard in combine step

## Output Dataset

Final panel `output/panel_data_public.csv`:
- `AGS` - 8-digit municipality code (2021 standardized)
- `year` - 2005-2021
- `share_broadband_baseline` - >=0.128 Mbps (2005-2009) / >=1 Mbps (2010+)
- `share_gte1mbps`, `share_gte6mbps`, `share_gte30mbps` - Speed tiers
- `method_change_2015` - Dummy for 2015 methodological break

### Data Quality Notes

Recent fixes (Feb 2026):
- **2005-2008 baseline**: now correctly uses `speed_mbps_gte == 0` for historical DSL (~82% mean coverage)
- **AGS validation**: ~1,388 unmapped AGS codes filtered (see `docs/unmapped_ags_documentation.md`)
- **Value bounds**: all coverage values in [0, 100]

## Common Modifications

### Adding a new data source
1. Create `process_paket_X.R` following existing patterns
2. Extend `parse_broadband_variable()` for new naming conventions
3. Output long format: AGS, year, data_category, technology_group, speed_mbps_gte, value, original_variable
4. Update `05_combine_datasets.R` to include new Paket
5. Re-run steps 05-07

### Updating Destatis crosswalk
1. Replace files in `data/muni_mergers/`
2. Update `process_merger_sheet()` call in `06_standardize_ags_to_2021.R`
3. Column detection is dynamic via `str_detect` patterns

## R Dependencies

Primary: `tidyverse`, `readxl`, `stringr`, `here`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hhilbig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
