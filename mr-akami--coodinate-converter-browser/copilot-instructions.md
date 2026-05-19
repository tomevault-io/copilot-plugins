## coodinate-converter-browser

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Browser-based coordinate transformation using PROJ compiled to WebAssembly. proj-data stored in OPFS (Origin Private File System) for offline use after initial load. Chrome-targeted PoC.

## Dev Environment

Requires Nix:
```bash
nix develop          # enter dev shell (emscripten, cmake, ninja, nodejs, etc.)
```

PROJ is a git submodule:
```bash
git submodule update --init --recursive
```

## Build

```bash
# full build (sqlite3 + zlib + libtiff + PROJ + wasm bundle)
./scripts/build-proj-wasm.sh

# skip optional deps
WITH_TIFF=0 WITH_ZLIB=0 ./scripts/build-proj-wasm.sh

# force rebuild everything
FORCE_REBUILD=1 ./scripts/build-proj-wasm.sh
```

Output: `dist/proj_wasm.js` + `dist/proj_wasm.wasm` (ES6 module, MODULARIZE=1)

## Architecture

**Build chain:** Emscripten cross-compiles SQLite3, zlib, libtiff, then PROJ as static libs. `src/proj_wasm.c` wraps PROJ's C API into 4 exported functions, linked into final wasm bundle by `emcc`.

**WASM API** (`src/proj_wasm.c`):
- `proj_init(data_dir)` — init context, set search paths + DB
- `proj_transform(src, dst, x, y, z)` — coordinate transform between CRS IDs
- `proj_clear_cache()` — flush cached projection operation
- `proj_cleanup()` — destroy context

Caches last-used projection (src/dst pair) to avoid recreation on repeated transforms.

**Data flow:** Hono server serves `proj.db` + manifest at `/api/proj-data/manifest`, individual grids at `/api/proj-data/v/<version>/grids/<name>`. Frontend downloads manifest + proj.db only at startup (~10MB), caches in OPFS. On each transform, `pw_grids_needed` enumerates required grids; missing ones are fetched lazily, sha256-verified, mounted into MEMFS, then `pw_refresh_after_grid_write` recreates the PROJ context to invalidate availability caches. `pw_transform` is **strict**: it refuses to silently fall back to a less-accurate Helmert/ballpark op when a grid is missing — instead it throws `MissingGridError` so the caller can decide.

**Hono server:** `npm run build:manifest` (build-time) generates `server/manifest.json` from `third_party/sc-proj-data/proj/`. Allow-list = (`grid_alternatives.{proj_grid_name, original_grid_name, old_proj_grid_name}` ∪ `grid_transformation.grid_name` ∪ `other_transformation.grid_name`) ∩ on-disk files. `npm run dev:server` runs the server on port 3000.

## Key Files

- `src/proj_wasm.c` — C wrapper exposing PROJ to JS
- `scripts/build-proj-wasm.sh` — full build orchestration (~240 lines)
- `agent.md` — project goals/architecture (Japanese)
- `docs/proj-wasm-build.md` — build details and API notes
- `third_party/proj/` — PROJ submodule

---
> Source: [Mr-akami/coodinate-converter-browser](https://github.com/Mr-akami/coodinate-converter-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
