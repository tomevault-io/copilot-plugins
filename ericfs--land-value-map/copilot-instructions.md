## land-value-map

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This project computes and maps land value per acre for every parcel in Connecticut, using data from the [2024 Connecticut Parcel and CAMA Data](https://data.ct.gov/Local-Government/2024-Connecticut-Parcel-and-CAMA-Data/pqrn-qghw/about_data). The output is a vector tile map served at http://localhost:8080 (or https://map.strongerhaven.org in production).

## Build

**Prerequisites:** Copy the example config files and fill in your API keys:
- `ansible/example.config.yml` → `ansible/config.yml` (shared settings)
- `ansible/example.dev.yml` → `ansible/dev.yml` (local dev: localhost, gzipped tiles)
- `ansible/example.prod.yml` → `ansible/prod.yml` (production: strongerhaven.org, no gzip)

```sh
# From project root — runs inside dev container, or builds container first if outside
./scripts/build.sh          # defaults to dev
./scripts/build.sh dev      # explicit dev build
./scripts/build.sh prod     # production build
```

Inside the dev container, this runs the Ansible playbook directly:
```sh
ansible-playbook ansible/main.yml -e "working_dir=$(pwd) env=dev"
```

To run the Python processing step alone:
```sh
cd src
python3 main.py --input_dir=inputs --output_dir=tmp
```

## Architecture

The pipeline has four stages orchestrated by `ansible/main.yml`:

1. **GeoJSON generation** (`src/`): Python scripts read GDB parcel files and CAMA CSV files, join them, compute value per acre, and export per-town GeoJSON files to `tmp/geojson/`.

2. **Tile generation** (`tippecanoe/gen_tiles.sh`): Runs `tippecanoe` on all GeoJSON files to produce vector map tiles (`.pbf`) in `tmp/tiles/`. At lower zoom levels, tiles accumulate `Appraised_Total` and `Land_Acres` so value-per-acre can be computed client-side for coalesced parcels.

3. **tile.json generation** (`src/merge_tilejson.py`): Merges `templates/tile.json` with tippecanoe metadata to produce `tmp/tile.json` with correct bounds, zoom levels, and tile URL template.

4. **Deployment** (`serving/`): Ansible syncs the static site and tiles to `deploy_dir`, writes `templates/index.html.j2` to HTML, and starts/reloads nginx.

### Key Source Files

- `src/main.py` — Entry point; parallelizes GDB processing across CPUs
- `src/cog.py` — Processes one COG (Council of Governments) GDB; orchestrates per-town join and export
- `src/town_layers.py` — Matches CAMA and Parcel layers within a GDB by name pattern
- `src/town_join.py` — **Core challenge**: joins Parcel geometry with CAMA data; uses metadata join keys with heuristic fallbacks and validation
- `src/value_per_acre.py` — Computes `Appraised_Value_Per_Acre = Appraised_Total / Land_Acres` and caps at 90th percentile
- `src/export_geojson.py` — Reprojects to EPSG:4326 and exports GeoJSON for tippecanoe; filters parcels < 0.02 acres
- `src/merge_tilejson.py` — Standalone script to merge tile metadata into tile.json template

### Input Data Structure

```
inputs/
  Parcel Collection 2024/
    Parcel_By_COG/<COG_NAME>/<COG_NAME>.gdb   # GDB with parcel geometry + CAMA layers
    CAMA_By_COG/<COG_NAME>/<Town>_2024_CAMA.csv
  Metadata_2024.csv   # Per-town join field names (often incorrect — see agent/notes.md)
```

Each GDB contains layers named `<Town>_Parcels` and `<Town>_2024_CAMA`. The join between them is the core difficulty (see `agent/notes.md` for known problematic towns).

### Config (`ansible/`)

Config is split into a shared base and per-environment overrides:
- `config.yml` — Shared settings (input paths, build dirs)
- `dev.yml` — Dev overrides (localhost, gzipped tiles for nginx `gzip_static`)
- `prod.yml` — Prod overrides (strongerhaven.org, no tile gzip)

Key variables:
- `maptiler_api_key` — Required for the map frontend
- `build_dir` — Intermediate output (default: `tmp/`)
- `deploy_dir` — Where the built site is deployed (default: `~/tmp/landvalue`)
- `hostname` / `scheme` / `version` — Used to generate tile URLs in tile.json
- `gzip_tiles` — Whether to pre-gzip `.pbf` tiles (true for dev, false for prod)

### Dev Container

The `.devcontainer/Dockerfile` builds an image with tippecanoe (compiled from source), Python geo libraries (`geopandas`, `fiona`, `pyogc`, `contextily`), Ansible, and nginx. The dev container is also the build environment used by `scripts/build.sh`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ericfs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
