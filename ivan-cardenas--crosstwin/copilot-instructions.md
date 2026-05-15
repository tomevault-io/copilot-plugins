## crosstwin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CrossTwin is a Django-based digital twin platform for geospatial urban data (water supply, urban heat, energy, housing, nature). It uses PostgreSQL/PostGIS for spatial storage, Mapbox GL JS for map rendering, and TiTiler (FastAPI) for raster tile serving.

The project focuses on **Dutch geospatial data** — the default CRS is EPSG:28992 (Amersfoort / RD New), configured via `COORDINATE_SYSTEM` in `.env`.

## Common Commands

```bash
# Activate virtual environment (Windows)
.venv\Scripts\Activate

# Start both Django and TiTiler (or use start.bat)
python manage.py runserver 8000
uvicorn tiler:app --port 8001 --reload

# Database migrations
python manage.py makemigrations
python manage.py migrate

# Run all tests (uses custom PostGIS test runner)
python manage.py test --settings=DigitalTwin.settings_test

# Run tests for a single app
python manage.py test importer --settings=DigitalTwin.settings_test

# Export COGs for raster models
python manage.py export_cogs
```

## Architecture

### Django Settings & Two-Server Setup

- **Django project**: `DigitalTwin/` — settings load from `.env` via `python-dotenv`
- **TiTiler**: `tiler.py` — a separate FastAPI app serving COG raster tiles on port 8001. Django proxies to it via `TITILER_BASE_URL` env var.
- Windows GDAL/GEOS DLLs are auto-discovered from the venv's `osgeo` package in `settings.py`.

### Model Registry System (`core/utils.py`)

The central architectural pattern. `build_model_registry()` scans a hardcoded list of allowed apps (`common`, `urbanHeat`, `watersupply`, `weather`, `builtup`, `Energy`, `Housing`, `nature`) and builds four registries:

- **`MODEL_REGISTRY`** — all models from allowed apps
- **`VECTOR_REGISTRY`** — models with GeometryField (no RasterField)
- **`RASTER_REGISTRY`** — models with RasterField
- **`WMS_REGISTRY`** — models with "WMS" in the registry key

These registries power generic API endpoints, the map layer catalog, and the importer. When adding a new domain app, it must be added to the `allowed_apps` list in `core/utils.py` and to `INSTALLED_APPS` in `settings.py`.

### DPSIR Causal Network (`common/DAG.dot`)

The project follows the **DPSIR framework** (Driver → Pressure → State → Impact → Response) to model causal relationships between urban systems. The full causal graph is defined in `common/DAG.dot` (Graphviz format). Each domain app's `calculations.py` documents which DAG edges its functions implement via comments like `# DAG edges: Central_Bank -> Mortgage`.

When adding new calculations, trace the relevant DAG edges and document them in the function docstring to maintain traceability between the causal model and code.

### Domain Apps

Each domain app contains spatial models related to a topic:

- **`common`** — Administrative hierarchy: Province > City > District > Neighborhood. Also: LandCover, DEM, DSM, material properties, environmental costs. Population cascades upward via signals.
- **`watersupply`** — Water infrastructure: extraction, treatment, pipe networks, coverage, NRW (non-revenue water), OPEX. Has `calculations.py` (pure query functions) + `views.py` (indicator assembly + HTMX recalculation).
- **`urban_heat`** — Thermal comfort rasters (UTCI, PET, MRT, LST, SVF, SUHII) and Nature-Based Solutions.
- **`housing`** — Supply/demand, mortgages, rentals, HPI, affordability stress. Has `calculations.py` + `views.py` following the same pattern as watersupply.
- **`builtup`** — Physical urban fabric: ZoningArea, Street, Park, Facility, Building, Property.
- **`nature`**, **`weather`**, **`Energy`** — Additional domain data.

### Indicator Pattern (watersupply, housing)

Domain dashboards follow a three-layer pattern:
1. **`calculations.py`** — Pure query functions that aggregate DB data. Each function documents its DAG edges.
2. **`views.py`** — `_get_province_data()` assembles all calculations into a dict; `_build_indicators()` is a pure function that derives display-ready metrics (supports what-if overrides like `consumption_override` or `interest_rate_override`); view functions render templates or return JSON.
3. **Templates** — HTMX-driven: a main page includes a partial grid that gets swapped on slider/input changes via `hx-get` to a `recalculate_indicators` endpoint.

`MOCK_DATA` dicts provide fallback values when the province doesn't exist in DB, allowing frontend development without a populated database.

### Signal-Driven Computations

- **`common/signals.py`** — When a Neighborhood is saved/deleted, population and density cascade up through District > City > Province using `_recompute_population()`. Uses `update()` (not `save()`) to avoid infinite loops.
- **`core/signals.py`** — `post_save` on every RASTER_REGISTRY model auto-exports to COG via `export_raster_to_cog()`.

### Importer System (`importer/`)

Two import paths:
1. **File upload** (`views.py`) — Upload GeoJSON/Shapefile, map fields to any registered model, preview, then import with savepoints. Uses `MODEL_OVERRIDES` dict for per-model upsert keys.
2. **External data** (`views_external.py`, `external_catalog.py`, `external_data.py`) — Catalog-driven import from PDOK, CBS, Sentinel-2, Google Earth Engine.

### Raster Pipeline (`core/rasterOperations.py`)

1. PostGIS raster → `ST_AsGDALRaster` → temp GeoTIFF
2. Reproject to EPSG:4326 via rasterio
3. Convert to Cloud-Optimized GeoTIFF (COG) via `rio-cogeo`
4. COG stored on disk under `cogs/<app_label>/`
5. TiTiler serves tiles from the COG path

### Map Frontend (`mainMap/`)

- `map_view` renders `mainMap.html` with Mapbox token
- `available_layers` returns the full layer catalog (vector + WMS + raster) as JSON
- `model_geojson` uses raw SQL with `ST_AsGeoJSON(ST_Transform(..., 4326))` for any vector model
- Templates live in `Templates/` (capital T, configured in settings)

### API Routes

| Path | Purpose |
|---|---|
| `/` | Main map view |
| `/map/api/layers/` | Layer catalog JSON |
| `/api/layers/<app>/<model>/geojson/` | GeoJSON for vector model |
| `/api/layers/<app>/<model>/bounds/` | Bounding box extent |
| `/api/raster/<app>/<model>/tiles/` | TiTiler tile URL for raster |
| `/api/raster/<app>/<model>/info/` | Raster metadata |
| `/importer/` | File upload import |
| `/common/` | External data import + CBS importer |
| `/watersupply/` | Water supply indicators dashboard |
| `/urban_heat/` | Urban heat views |
| `/housing/` | Housing indicators dashboard |

### Testing

Tests use `PostGISTestRunner` (`DigitalTwin/test_runner.py`) which creates the test DB, installs PostGIS extensions, and patches Django's `prepare_database` to avoid superuser requirement. Always use `--settings=DigitalTwin.settings_test`.

## Key Patterns to Follow

- All spatial models use `srid=settings.COORDINATE_SYSTEM` (EPSG:28992), not hardcoded SRIDs
- Models with computed fields implement logic in `save()` — check existing patterns before adding new ones
- Use `models.DO_NOTHING` for FK on_delete in spatial/reference models (project convention)
- Population cascading uses `update()` not `save()` to prevent signal recursion
- Raster models need a `cog_path` field to participate in the COG auto-export pipeline
- The GeoJSON API transforms to EPSG:4326 at query time via `ST_Transform`
- Frontend CSS uses Tailwind with crispy-tailwind for forms

## Known Issues

- **Registry case mismatch**: `core/utils.py` lists `'Housing'` (capital H) in `allowed_apps`, but the Django app name is `'housing'` (lowercase in `apps.py` and `INSTALLED_APPS`). Housing models won't appear in `MODEL_REGISTRY` or as map layers until this is fixed. The `calculations.py` and `views.py` work via direct imports and are unaffected.

## TODOs

Collected from inline `#TODO` comments across the codebase:

### core
- Add H3 hexagon configuration for spatial aggregation (`core/utils.py`)

### common
- Auto-calculate `LandCoverVector.percentage` from geom area vs Province total area (`common/models.py`)
- Add Vegetation Coverage and Builtup Coverage as additional fields on `LandCoverVector` (`common/models.py`)

### builtup
- Define connectivity index and calculation method for `Building.connectivity` (`builtup/models.py`)
- Define green visibility index and calculation method for `Property.greenVisibility` (`builtup/models.py`)

### watersupply
- `UsersLocation.neighborhood` FK — decide whether this should be City-level or per-point (`watersupply/models.py`)
- `AvailableFreshWater.infiltrationRate_cm_h` — should be calculated from land cover and soil type (`watersupply/models.py`)
- `ExtractionWater.pumpEmissionRate_kg_CO2_h` — should be auto-calculated from energy rate and electricity emission factor (`watersupply/models.py`)
- `TotalWaterProduction.source` — handle imported or multiple sources (`watersupply/models.py`)
- `TotalWaterProduction.save()` — set a proper default or handle missing `ElectricityCost` (`watersupply/models.py`)

### housing
- Validate affordability stress level thresholds against literature (`housing/models.py`)
- Connect `HousingAffordability.medianExpenditure` with water and electricity expenditure data (`housing/models.py`)
- Decide whether affordability index should use median disposable income instead of median income (`housing/models.py`)

### urban_heat
- Add measurement method, source, and metadata fields to raster models: MRT, UTCI, SVF, LST (`urban_heat/models.py`)

### Energy
- Populate `EnergyEfficiencyLabels` with standard descriptions and connect to labels (`Energy/models.py`)

---
> Source: [ivan-cardenas/CrossTwin](https://github.com/ivan-cardenas/CrossTwin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
