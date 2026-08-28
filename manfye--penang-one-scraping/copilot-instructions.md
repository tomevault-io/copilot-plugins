## penang-one-scraping

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Two parts:
1. A scraper (`fetch.py`) for Penang state government open-data ArcGIS FeatureServer layers: EV chargers, AEDs (defibrillators), and CCTV cameras across Penang. Writes raw JSON + flattened CSV to `output/`.
2. `web/` — a mobile-first Next.js app ("Terdekat") that lets a user find the nearest EV charger / AED / CCTV to their current location and hand off to Google Maps, Waze, or OpenStreetMap for actual navigation.

## Commands

Re-scrape the source data (overwrites all files in `output/`):

```bash
python3 fetch.py
```

Convert the raw scrape into the slim JSON the web app consumes (run after `fetch.py` whenever source data changes):

```bash
python3 scripts/convert-data.py
```

Web app (run from `web/`):

```bash
npm run dev      # dev server
npm run build    # production build
npx tsc --noEmit # typecheck
npx eslint .      # lint
```

No test suite is set up.

## Architecture

`fetch.py` is a single script with one pattern repeated for three layers, defined in the `layers` dict:

```python
layers = {
    "cctv": ("CCTV2026", 0),
    "aed": ("AED_PENANG", 0),
    "ev_charger": ("fl_EV100326", 0),
}
```

Each entry maps an output name to `(service_name, layer_index)` on `https://pegis.penang.gov.my/arcgis/rest/services/Hosted`. `fetch_all()` pages through the ArcGIS `query` REST endpoint 2000 records at a time (`resultOffset`/`resultRecordCount`) until a short page signals the end. To add a new layer, add an entry to `layers` — no other code changes needed.

For each layer, output is written twice:
- `output/<name>.json` — the raw ArcGIS feature list (`attributes` + `geometry` per feature), indented.
- `output/<name>.csv` — flattened: `attributes` fields plus `geometry_x`/`geometry_y` pulled out of `geometry`, using the first feature's attribute keys as the CSV header (assumes all features in a layer share the same schema).

### Data shape notes

- Coordinates: each feature's `geometry.x`/`geometry.y` are lon/lat (EPSG:4326, since `outSR=4326` is requested). Most layers also duplicate this in attributes as `lat`/`long`, and EV chargers additionally have a human-readable `location_coordinate` string (`"lat, long"`).
- Field names are a mix of Malay and English and vary per layer (EV charger, AED, and CCTV each have distinct attribute schemas) — there is no shared schema across `output/*.json` beyond `attributes`/`geometry`.
- Nullable fields are common (e.g. `remark`, `pic`, `catatan`); don't assume required fields are populated.
- **Each row is a physical site, not a unit.** A site can host multiple individual units. The official Penang OneMap+ portal's headline totals (e.g. "1,977 CCTV") count units, not sites — verified by summing the per-site quantity field: `jumlah_camera1` for CCTV (678 sites → 1,977 cameras), `jumlah` for AED (427 sites → 439 devices), `jumlah_bilangan_unit_pengecas` for EV chargers (225 sites → 494 charge points; note this is the field that reconciles with the portal, not `jumlah_bay`). `scripts/convert-data.py` captures this as `unitCount` on each converted record.

## `scripts/convert-data.py`

Reads `output/*.json`, keeps only the fields the web app needs (id, category, name, address, lat/lng, `unitCount`, a small `extra` bag of category-specific fields), drops any feature missing coordinates, and writes one array per category to `web/public/data/*.json` plus a `manifest.json` with `{sites, units}` counts per category. Add a new layer by adding both a `fetch.py` entry and a `convert_<name>` function + `LAYERS` entry here.

## `web/` — the Next.js app

App Router, TypeScript, Tailwind v4. Single-page, mobile-first "navigator" UI (`src/app/page.tsx`) — no routing beyond `/`, no backend/API routes, no database. All location data is static JSON fetched client-side from `public/data/`.

Architecture:
- `src/lib/types.ts` — `LocationItem` (as converted) / `LocationItemWithDistance` (item + computed distance & bearing from the user).
- `src/lib/categories.ts` — per-category metadata (emoji, labels, data file path, literal Tailwind class strings). Classes are spelled out in full per category rather than built via string interpolation, because Tailwind's JIT scanner can't see dynamically-constructed class names like `` `bg-${accent}-500` ``.
- `src/lib/geo.ts` — haversine distance, initial compass bearing, distance/compass-point formatting.
- `src/lib/navLinks.ts` — builds the three navigation hand-off URLs (Google Maps, Waze, OpenStreetMap directions) for a destination; this is the "use OSM if the user wants to navigate" option, not in-app routing.
- `src/hooks/useGeolocation.ts` — wraps `navigator.geolocation.watchPosition`.
- `src/hooks/useHeading.ts` — device compass heading via `deviceorientation`; handles iOS's `DeviceOrientationEvent.requestPermission()` gate separately (`needsPermission` + `enable()`).
- `src/hooks/useNearby.ts` — fetches a category's JSON once (module-level `Map` cache shared across the app, not React state, to avoid re-fetching on tab switches), then derives distance-sorted results from geolocation.
- `src/components/MapView.tsx` — Leaflet map (OSM raster tiles), dynamically imported with `ssr: false` in `page.tsx` since Leaflet touches `window` at import time.
- `src/components/NavSheet.tsx` — bottom sheet offering the three nav hand-off links for a selected item.

The list view is the primary UI (per product direction: mobile-first "navigator", map is secondary); the List/Map toggle lives in the header.

---
> Source: [manfye/penang_one_scraping](https://github.com/manfye/penang_one_scraping) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
