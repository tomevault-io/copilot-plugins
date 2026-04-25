## parkpilotsd

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ParkPilotSD** is a single-file React dashboard (`ParkPilotSD.jsx`) for analyzing San Diego parking meter data. It fetches live CSVs from the City of San Diego's open data portal and renders interactive analytics across 8 tabs.

## Architecture

Everything lives in one monolithic component (`ParkPilotSD.jsx`). There is no build system, package.json, or test suite — this is likely embedded in an external React app (e.g., a Claude artifact or CodeSandbox). Dependencies: `papaparse` for CSV parsing, `recharts` for all charts.

### Data Flow

1. **Load phase** (`loadAll` callback): fetches 5 CSVs in sequence — meter locations + monthly transaction summaries for 2022–2025 — from `seshat.datasd.org`. Falls back to user file uploads if remote fetch fails.
2. **Analytics** (`useMemo` on `[locations, transactions]`): normalizes column names from both CSV schemas, joins meters to transactions by `pole_id`, and computes all derived metrics (zone stats, monthly trends, busyness, top/bottom earners, config distribution).
3. **Render**: tab-based UI, each tab reads from the `analytics` object. `filteredMeters` drives the Directory and Find Parking tabs with search/filter/sort.

### CSV Data Schemas

| File | Key columns |
|---|---|
| `parking_meters_current.csv` | zone, area, sub-area, pole, lat, lon, configid, configname, price, days_in_operation |
| `treas_parking_meters_loc_datasd.csv` | zone, area, sub_area, pole, config_id, config_name, start_date, end_date |
| `treas_parking_payments_YYYY_datasd.csv` | pole_id, meter_type, date_trans_start, date_meter_expire, trans_amt, pay_method |

The analytics layer uses fuzzy column-name matching (lowercased, non-alphanumeric stripped) to handle schema variations across CSV versions.

### Tabs

| Tab ID | Purpose |
|---|---|
| `overview` | KPI cards + zone revenue bar chart + monthly trend line + config pie |
| `directory` | Filterable/sortable table of all meters |
| `finder` | Find Parking — filtered meter cards with revenue/spend/transaction stats |
| `trends` | Monthly revenue and transaction count area chart (2022–2025) |
| `spend` | Average spend per transaction scatter/bar |
| `leaderboard` | Top and bottom earning meters |
| `busyness` | Busiest meters by transaction count |
| `free` | Free/cheap meter finder (zero avg_spend or lowest avg_spend) |

### Color Palette

Defined in the `C` constant (dark theme: `#0a0e1a` background, `#00d4ff` accent cyan). Zone colors use `ZONE_COLORS` array. All inline styles reference these constants.

## Local CSV Files

The repo contains reference/offline copies of the data:
- `parking_meters_current.csv` — current meter config with lat/lon
- `treas_parking_meters_loc_datasd.csv` — historical meter location configs
- `treas_parking_payments_2019–2026_datasd.csv` — yearly payment records
- `city_property_parcels_datasd.csv` — city parcel data (APN mapping)

The live app fetches monthly-aggregated transaction CSVs from `seshat.datasd.org`; the local yearly files contain raw per-transaction records with different schema (`pole_id`, `trans_amt`, `pay_method`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/avinash-gudipati) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
