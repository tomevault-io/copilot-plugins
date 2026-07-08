## aquiferx

> - React 19 + TypeScript 5.8 + Vite 6 groundwater data visualization app

# Aquifer Analyst

## Project Overview
- React 19 + TypeScript 5.8 + Vite 6 groundwater data visualization app
- Data stored in `public/data/{region-folder}/` with per-folder `region.json`
- Vite dev server middleware provides API endpoints in `vite.config.ts` (no separate backend)
- Tailwind CSS for styling, Recharts for charts, Leaflet for maps

## Architecture
- **Hub-and-spoke data management**: `components/import/ImportDataHub.tsx` is the entry point, launching sub-wizards for Region, Aquifer, Well, and Measurement imports
- **Data type model**: `public/data/catalog_wq.json` is the global vocabulary of standard water quality parameters. Every region implicitly has every catalog entry — a catalog type "exists" in a region only when its `data_{code}.csv` is present on disk. Region-specific non-catalog parameters (BOD5, trichloroethane, etc.) live in `region.json` as `customDataTypes`. `effectiveDataTypes` (WTE + catalog-with-data + custom-with-data) is computed at load time via `services/catalog.ts#computeEffectiveDataTypes`.
- **Measurements**: use `value` field + `dataType` code. Per-file: `data_{code}.csv` naming (e.g. `data_wte.csv`, `data_nitrate.csv`)
- **Per-folder metadata**: Each region folder has `region.json` with id, name, lengthUnit, singleUnit, customDataTypes
- **CRS reprojection**: proj4 library in `services/reprojection.ts`, auto-detects from GeoJSON `crs` property or shapefile `.prj`
- **USGS API**: `services/usgsApi.ts` for downloading wells and measurements from USGS Water Data API
- **Smart well discovery**: `services/wellMatching.ts` provides ID → name → proximity matching for CSV imports, plus per-column catalog-aware data type detection and `aqx-` ID generation for new wells. `services/gseLookup.ts` batches USGS 3DEP / Open-Meteo elevation fetches.

## Key Files
- `App.tsx` — main component, state management, data type selector
- `types.ts` — DataType, RegionMeta (customDataTypes), Region (effectiveDataTypes), CatalogParameter, ParameterCatalog, Measurement, ChartPoint interfaces
- `services/dataLoader.ts` — loads regions via `/api/regions`, computes effectiveDataTypes, loads measurements from each `data_{code}.csv`
- `services/catalog.ts` — `loadCatalog()`, `computeEffectiveDataTypes()`, `groupCatalog()`
- `services/wellMatching.ts` — matchWells (ID/name/proximity), generateAqxId, suggestDataTypesFromColumns
- `services/gseLookup.ts` — batched elevation lookup (USGS 3DEP / Open-Meteo)
- `services/importUtils.ts` — CSV parsing, file processing, point-in-polygon, save/delete API wrappers
- `services/reprojection.ts` — proj4 coordinate reprojection
- `services/usgsApi.ts` — USGS Water Data API integration
- `components/CatalogBrowser.tsx` — read-only catalog viewer, opened from DataTypeEditor and MeasurementImporter
- `components/import/` — ImportDataHub, RegionImporter, AquiferImporter, WellImporter, MeasurementImporter, DataTypeEditor, ColumnMapperModal, ConfirmDialog
- `public/data/catalog_wq.json` — global water quality parameter catalog (~38 entries with WQP mapping + MCL/WHO)
- `vite.config.ts` — API middleware endpoints (regions, save-data, delete-file, delete-folder). `/api/regions` scans each region folder and returns a `dataFiles: string[]` list alongside the meta.

## Conventions
- Data type codes: lowercase alphanumeric + underscore, max 20 chars; "wte" is the reserved default (water table elevation)
- **Catalog is authoritative**: standard WQ parameters (nitrate, arsenic, pH, etc.) live in `catalog_wq.json` and cannot be overridden per-region. If a region needs a different unit or name for a parameter, make a custom type with a non-catalog code (e.g. `hardness_caco3`) — not a rename of the catalog entry
- **No pre-staging empty types**: a data type exists in a region exactly when its `data_{code}.csv` exists on disk. There is no way to "declare a type in advance" — importing data is the only action that creates one. DataTypeEditor only manages non-catalog customs; even those must have a data file to appear in the header dropdown
- **Custom codes must not collide with catalog codes**: DataTypeEditor and the importer's detection panel both enforce this
- Single-unit regions: aquifer section dimmed in UI, all data auto-assigned `aquifer_id=0`
- CSV delimiter auto-detection (comma vs tab)
- Date format auto-detection (ISO, US, EU variants)
- Measurement values stored in `value` column (not `wte`)
- Region data lives in `public/data/{region-id}/` with: `region.json`, `region.geojson`, `aquifers.geojson`, `wells.csv`, `data_{code}.csv`

## Commands
- `npm run dev` — start dev server on port 3000
- `npx tsc --noEmit` — type check
- `npx vite build` — production build

---
> Source: [njones61/aquiferx](https://github.com/njones61/aquiferx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
