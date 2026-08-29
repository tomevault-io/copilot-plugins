## wm-land-screener

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

West Michigan vacant land screener. Pulls parcel, zoning, floodplain, and wetland data from public APIs, overlays them spatially, and scores/filters parcels for development feasibility. Phase 1 targets Grand Haven; Phase 4 expands to Holland and Muskegon.

## Commands

```bash
# Launch the Streamlit web UI (opens at http://localhost:8501)
cd "MLS Tool Experiment" && /Users/sadiebrooks/Library/Python/3.9/bin/streamlit run src/app.py

# Run full pipeline (uses cached data if available)
cd "MLS Tool Experiment" && python3 src/pipeline.py

# Force fresh download of all layers
python3 src/pipeline.py --refresh

# Single city only
python3 src/pipeline.py --city grand_haven

# Install dependencies
pip3 install -r requirements.txt
```

## Architecture

```
MLS Tool Experiment/
├── src/
│   ├── config.py        # All constants: URLs, thresholds, per-city min_acres, zoning lookup table
│   ├── data_loader.py   # Downloads from ArcGIS REST, FEMA, NWI, OSM Overpass — caches to data/raw/
│   ├── overlay.py       # GeoPandas spatial joins: zoning, flood %, wetland %, building coverage
│   ├── scoring.py       # Hard filters + 0-100 soft score per parcel
│   ├── pipeline.py      # Orchestrates all steps, writes output/ CSVs + GeoJSON
│   └── app.py           # Streamlit web UI — interactive map + filters + table
├── data/
│   ├── raw/             # Cached GeoJSON downloads (safe to delete to re-fetch)
│   └── processed/       # (reserved for future processed layers)
└── output/              # grand_haven_all_parcels.csv + qualified_parcels.csv + .geojson
```

## Key Data Sources

All confirmed working and cached:

| Layer | Service URL / Source |
|---|---|
| Grand Haven Parcels | `services2.arcgis.com/nPodhnBI3xr2aJGu/.../GH_Zoning_Map_WFL1/FeatureServer/8` |
| Grand Haven Zoning (zZoning2020) | `…/FeatureServer/12` — fields: `ZONE_` (code), `ZONE1` (description) |
| FEMA Flood Zones | `hazards.fema.gov/arcgis/rest/services/public/NFHL/MapServer/28` |
| NWI Wetlands | `fwspublicservices.wim.usgs.gov/wetlandsmapservice/…/Wetlands/MapServer/0` |
| OSM Buildings | Overpass API — `overpass-api.de/api/interpreter` (way+relation `["building"]`) |
| **Ottawa County Master Plan** | **Live service** — `gis.miottawa.org/arcgis/rest/services/HostedServices/MasterPlanZoning/FeatureServer/2`. Covers Grand Haven city + township, Spring Lake, Ferrysburg. `Stan_Class` field used (county-standardized). Caches to `data/raw/grand_haven_future_lu.geojson`. |

## Important Field Notes

- **Zoning**: `ZONE_` = short code (MDR, MFR, LDR, etc.), `ZONE1` = full description. Density lookup in `config.py` uses `ZONE_` keys.
- **Parcels**: `CLASS` field uses Michigan assessor codes (401=all residential, 201=commercial). **Not usable for vacancy detection** — all parcels share the same class. Vacancy is now detected via OSM building footprint coverage (`building_pct > 1%` = improved). Falls back to `SEVVALUE/acre < $150k` proxy if building data unavailable.
- **Per-city min_acres**: Set `"min_acres"` in each city's `CITIES` dict to override the global `MIN_ACRES = 4.0`. Grand Haven is set to `2.0` (dense urban area). Omit for new cities to get the 4-acre default.
- **Pagination**: ArcGIS service caps at 2000 records. `_arcgis_query()` in `data_loader.py` pages automatically via `exceededTransferLimit`.

## Zoning Lookup

`config.py:GRAND_HAVEN_ZONING` maps zone codes → `max_units_per_acre`. These are estimates — verify against Grand Haven Chapter 40 ordinance before using for actual projects. Key codes: `MFR`=22, `MDR`=12, `LDR`=6, `CB`=30, `NMU`=18, `PD`=20.

## Future Land Use (Master Plan) Integration

`config.py:GRAND_HAVEN_FUTURE_LU` maps FLU category names → `max_units_per_acre` equivalents. The pipeline adds these columns when FLU data is loaded:
- `future_lu_code` / `future_lu_label` / `future_max_units` — master-plan designation
- `rezoning_upside` (bool) — True when future density > current zoning density
- `rezoning_delta` (int) — units/acre gap (future − current)
- `pts_rezoning` (0–10) — bonus score component; 0 when no FLU data loaded

**Live and active**: Downloads automatically via `GH_MASTERPLAN_SERVICE` in config.py. Uses `stan_class` field (county-standardized). The `rezoning_delta` / `rezoning_upside` columns are corrected in `add_scores()` after `max_units_per_acre` is computed (FLU overlay runs first with a 0 baseline; scoring pass fixes it).

**Risk interpretation**: negative `rezoning_delta` (current zoning > master plan) = potential downzoning risk. `Parks, Recreation, Natural Areas` in `future_lu_label` = city long-term vision opposes residential use even if current zoning allows it.

**Phase 4 expansion**: The Ottawa County service already covers Grand Haven Township, Spring Lake Township, and Ferrysburg — no new service URL needed when adding those cities to `CITIES`.

## Phase Roadmap

See `West_Michigan_Land_Tool_Roadmap.docx` for full plan. Next phase: MLS API integration (Phase 2) — RESO Web API via **SWMRC** OAuth credentials (note: roadmap doc still says GRAR/WLAR, update when Phase 2 starts).

---
> Source: [WR-SW-Dev/wm-land-screener](https://github.com/WR-SW-Dev/wm-land-screener) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
