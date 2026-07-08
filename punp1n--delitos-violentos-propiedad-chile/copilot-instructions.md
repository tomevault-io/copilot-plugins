## delitos-violentos-propiedad-chile

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Research project analyzing structural change in violent property crimes in Chile (2014–2024), using administrative police records (CCH), victimization surveys (ENUSC), and population estimates. The pipeline is modular: Python ETL → parquet assembly → R statistical modeling → LaTeX manuscript.

## Pipeline Execution Order

### Prerequisites
- SQL Server credentials must be in `data/SyJ/.env` (SQLSERVER_HOST, SQLSERVER_DATABASE, SQLSERVER_USER, SQLSERVER_PASSWORD)
- Python packages: `pandas`, `pyodbc`, `numpy`, `openpyxl`, `pyarrow`
- R packages: `dplyr`, `arrow`, `ggplot2`, `splines`, `sandwich`, `strucchange`, `lmtest`, `sf`, `survey`

### Stage 1: ETL (Python, run in order)
```bash
python paper1/etl/01_extract_cch.py           # SQL Server → cch_panel_comuna_month.parquet
python paper1/etl/02_extract_placebos.py       # SQL Server → placebo_panel.parquet
python paper1/etl/02b_build_cphdv.py           # CPHDV Excel → cphdv_homicidios.parquet
python paper1/etl/04_build_sermig.py           # SERMIG (imported by 03)
python paper1/etl/03_build_population.py       # INE 2017 + SERMIG → poblacion_regional_mensual.csv
python paper1/etl/03b_build_national_population.py  # INE 2024 → poblacion_nacional_mensual_base2024.csv
python paper1/etl/06_assemble_panel.py         # → panel_region_month.parquet (main analytical dataset)
```

### Stage 2: Statistical Modeling (R)
```bash
Rscript paper1/models/01_descriptive.R          # Descriptive stats + Fig 1–3
Rscript paper1/models/02_main_poisson_wcb.R     # Primary Poisson-QMLE + Wild Cluster Bootstrap
Rscript paper1/models/03_regional_cusum_fdr.R   # Regional CUSUM + Bai-Perron + Fig 4
Rscript paper1/models/05_cum_descriptive.R      # CUM code disaggregation
Rscript paper1/models/06_placebos.R             # Placebo models P1–P5 + Fig 5
Rscript paper1/models/07_robustness.R           # Robustness specs R1, R3, R5–R7
Rscript paper1/models/08_sensitivity_pop.R      # Population sensitivity (k = 1.00–1.20)
Rscript paper1/models/09_maps_figures.R         # Choropleth maps
```

### Stage 3: Manuscript
```bash
cd paper1/texto && pdflatex articulo.tex && bibtex articulo && pdflatex articulo.tex
```

## Architecture

### Key Data Flow
```
SQL Server (CCH) ──► 01_extract_cch.py ──► cch_panel_comuna_month.parquet
                                                        │
Excel (CPHDV, INE, SERMIG) ──► 03/04 scripts           │
                                    │                   ▼
                         poblacion_regional_mensual.csv ──► 06_assemble_panel.py
                                                                    │
                                                    panel_region_month.parquet
                                                    [16 regions × ~144 months]
                                                                    │
                                              ┌─────────────────────┼──────────────────────┐
                                           02_main.R          03_cusum.R           06_placebos.R
                                        (Poisson-WCB)     (CUSUM-FDR+BP)        (P1–P5 controls)
```

### Crime Classifications (run in parallel as sensitivity checks)
- **C1** – Official SPD/CAPJ definition (includes fencing)
- **C2** – C1 minus fencing (ICCS-aligned)
- **C3** – Tripartite: *Violencia Dura* / *Robo Sorpresa* / *No Violento* — **primary classification**

Output tables mirror this structure: `paper1/output/tables/C1/`, `/C2/`, `/C3/`

### Temporal Dummy Variables (defined in `06_assemble_panel.py`)
- `d_estallido` = Oct 2019 – Feb 2020 (social unrest period)
- `d_pandemia` = Mar 2020 – Dec 2021 (COVID-19 period)
- `trend_t` = months since Jan 2013

### Primary Model (`02_main_poisson_wcb.R`)
Poisson-QMLE with:
- Spline basis on `trend_t` with knots at percentiles P25/P50/P75 (not calendar dates)
- Controls: month FE, `d_estallido`, `d_pandemia`, region FE
- Offset: `log(pop_monthly)`
- Inference: Wild Cluster Bootstrap (R=9999, cluster=region)

### Placebo Tests (`06_placebos.R`)
- P1: Quasi-vehicular crimes (mobility proxy — should NOT spike)
- P2: National homicides (real violence — should spike if structural)
- P2b: CPHDV confirmed homicides (validation against dark figure)
- P3–P5: Kidnappings, simple damages, minor injuries

## Key Files
- `paper1/output/data/panel_region_month.parquet` — main analytical dataset consumed by all R models
- `data/SyJ/.env` — SQL Server credentials (never commit)
- `paper1/resultados/inferencial/interpretacion_resultados_v5.0.md` — latest interpretation of results
- `README.md` — full v4.0 methodology protocol

---
> Source: [punp1n/delitos_violentos_propiedad_chile](https://github.com/punp1n/delitos_violentos_propiedad_chile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
