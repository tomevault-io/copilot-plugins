## maa-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MAA Payment Record Management System — a Python/Streamlit app for tracking hospital admission insurance claims (MAA scheme). Uses SQLite for storage, pandas for data manipulation, and openpyxl for Excel report generation.

## Running the App

```bash
source .venv/bin/activate
streamlit run app.py
```

## Data Ingestion

```bash
# Ingest specific CSV files
python ingest.py file1.csv file2.csv

# Validate without writing (dry-run)
python ingest.py --dry-run

# Auto-detect GenericSearchReport*.csv from parent directory
python ingest.py
```

## Architecture

- **`app.py`** — Streamlit entry point. Page config, cached DB connection (`@st.cache_resource`), sidebar nav, page routing. 5 pages: Dashboard, Ingest, Admissions, Reports, Doctor Share.
- **`db.py`** — All database access. Schema: `claims` table (PK: `tid, pkg_code, claim_number`) + `claims_hash` for upsert change detection via MD5. Financial year starts April 1 (`fy_of()`).
- **`ingest.py`** — CSV parsing and ingestion. Handles unusual date formats (`parse_date_dmy`, `parse_payment_date`). Stores Aadhaar/ID/mobile numbers as text to preserve leading zeros.
- **`fetch.py`** — MAA portal scraper. Reuses an existing browser session (cookies) for fast fetches; falls back to Playwright for a full login flow. Writes CSV cache files for offline reuse.
- **`reports.py`** — In-memory `.xlsx` generation via openpyxl. Styled headers, color-coded rows (green=paid, amber=query), currency/date formatting, subtotals.
- **`utils.py`** — Shared helpers. `fmt_inr()` formats rupee amounts with Indian comma conventions.
- **`ui/`** — One module per page: `dashboard`, `ingest`, `admissions`, `reports`, `doctor_share`.

## Key Domain Details

- Currency is Indian Rupee (₹); use Indian number formatting conventions.
- Policy year and financial year are distinct fields. FY is April–March.
- Claims are uniquely identified by `(tid, pkg_code, claim_number)` — a patient (TID) can have multiple packages per admission.
- CSV source files are named `GenericSearchReport*.csv`.

---
> Source: [kartikeya23/maa-app](https://github.com/kartikeya23/maa-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
