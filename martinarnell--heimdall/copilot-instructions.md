## heimdall

> Compact geocoder built on FSTs. Nominatim-compatible API. Single Rust binary, no runtime dependencies. Currently supports 192 countries. 24 with official government address data (custom importers with CRS conversion where needed), 163 via Photon extracts, 5 via OSM PBF only (Pacific micro-states without Photon coverage). Per-country normalizer configs in `data/normalizers/` (66 files covering all language families) with language-specific phonetic encoding, script-aware normalization (Cyrillic transliteration, Arabic tashkeel stripping, CJK fullwidth conversion, Hebrew niqqud stripping, Thai tone mark handling, Devanagari matra stripping). Automated rebuild pipeline (`heimdall-build rebuild`) downloads sources, detects changes, and rebuilds affected indices.

# Heimdall — Development Guide

## What this is

Compact geocoder built on FSTs. Nominatim-compatible API. Single Rust binary, no runtime dependencies. Currently supports 192 countries. 24 with official government address data (custom importers with CRS conversion where needed), 163 via Photon extracts, 5 via OSM PBF only (Pacific micro-states without Photon coverage). Per-country normalizer configs in `data/normalizers/` (66 files covering all language families) with language-specific phonetic encoding, script-aware normalization (Cyrillic transliteration, Arabic tashkeel stripping, CJK fullwidth conversion, Hebrew niqqud stripping, Thai tone mark handling, Devanagari matra stripping). Automated rebuild pipeline (`heimdall-build rebuild`) downloads sources, detects changes, and rebuilds affected indices.

## Architecture

```
Manual:   OSM PBF → 3-pass extract → Parquet → enrich (admin hierarchy) → pack (FSTs + record stores)
                                                                         → pack_addr (street-grouped addresses)

Rebuild:  sources.toml → change detect → download → [extract → merge national → merge places_source → merge photon → enrich → pack]
          (per-country, wave-scheduled by RAM budget)
```

Query pipeline: normalizer → FST exact → FST phonetic → Levenshtein → fuzzy layers

Address pipeline: parse query → detect street+number+city → FST range scan → nearest to city coord

## Crate structure

```
crates/
  heimdall-core/       Types, FST index, record store, addr_store, reverse geocoding, NodeCache trait
  heimdall-build/      OSM extraction, enrich, pack, address pack, SSR GML parser, benchmark, rebuild pipeline
  heimdall-normalize/  Per-language text normalization, phonetic encoding, known_variants from TOML
  heimdall-nn/         Neural fuzzy layer stub (not implemented)
  heimdall-api/        Axum HTTP server, multi-country routing, Nominatim-compatible endpoints
  heimdall-compare/    Benchmark framework: 5 query categories, JSONL generation, SQLite results, reports (see crates/heimdall-compare/CLAUDE.md)
```

## Common commands

```bash
# Build everything
cargo build --release -p heimdall-build -p heimdall-api

# Build Sweden index (needs sweden-latest.osm.pbf in data/osm/)
cargo run --release -p heimdall-build -- build \
  --input data/osm/sweden-latest.osm.pbf \
  --output data/index-se

# Rebuild index (skip OSM extraction, reuse existing Parquet)
cargo run --release -p heimdall-build -- build \
  --input data/osm/sweden-latest.osm.pbf \
  --output data/index-se --skip-extract

# Run server (multi-country)
cargo run --release -p heimdall-api -- \
  --index data/index-se \
  --index data/index-no \
  --index data/index-dk \
  --index data/index-fi \
  --index data/index-de \
  --index data/index-gb \
  --index data/index-nz

# ── Benchmark vs Nominatim ─────────────────────────────────────────────

# Generate benchmark queries (5 categories, JSONL with metadata)
cargo run --release -p heimdall-compare -- generate-queries \
  --index data/index-se --index data/index-no \
  --count 10000 --seed 42 --output queries.jsonl

# Run benchmark (1 rps against public Nominatim, resumable)
cargo run --release -p heimdall-compare -- run \
  --queries queries.jsonl --rps 1 --output results.sqlite

# Generate report (console or markdown)
cargo run --release -p heimdall-compare -- report --db results.sqlite
cargo run --release -p heimdall-compare -- report --db results.sqlite --output accuracy.md

# Browse conflicts
cargo run --release -p heimdall-compare -- conflicts --db results.sqlite --min-distance 2000

# Continuous mode (legacy, samples from indices)
cargo run --release -p heimdall-compare -- continuous \
  --index data/index-se --rps 1

# Old benchmark commands (deprecated, use heimdall-compare instead)
cargo run --release -p heimdall-build -- gen-queries --index data/index-se --output queries.txt
cargo run --release -p heimdall-build -- bench --queries queries.txt

# Audit address data in a PBF
cargo run --release -p heimdall-build -- addr-audit --input data/osm/sweden-latest.osm.pbf

# Merge SSR place names into Norway (784K places from Kartverket)
cargo run --release -p heimdall-build -- merge-ssr \
  --index data/index-no \
  --gml data/norway/ssr_extract/Basisdata_0000_Norge_4258_stedsnavn_GML.gml

# Check index stats
cargo run --release -p heimdall-build -- stats --index data/index-se

# Import G-NAF addresses for Australia
cargo run --release -p heimdall-build -- gnaf-import \
  --index data/index-au \
  --gnaf-zip data/downloads/g-naf.zip

# Import NAR addresses for Canada
cargo run --release -p heimdall-build -- nar-import \
  --index data/index-ca \
  --nar-zip data/downloads/NAR_CAN.zip

# Import LINZ addresses for New Zealand
cargo run --release -p heimdall-build -- linz-import \
  --index data/index-nz \
  --linz-gpkg data/downloads/nz-street-address.gpkg

# ── Download sources ───────────────────────────────────────────────────

# Download all sources (up to 4 concurrent, no building)
cargo run --release -p heimdall-build -- download

# Download specific countries only
cargo run --release -p heimdall-build -- download --country dk,no,se

# Dry run — show what would be downloaded
cargo run --release -p heimdall-build -- download --dry-run

# Force redownload everything, ignoring change detection
cargo run --release -p heimdall-build -- download --redownload

# ── Automated rebuild ──────────────────────────────────────────────────

# Rebuild all countries (detect changes, download, build)
cargo run --release -p heimdall-build -- rebuild

# Rebuild from pre-downloaded sources (no network, use after `download`)
cargo run --release -p heimdall-build -- rebuild --skip-download

# Dry run — show what would change without downloading or building
cargo run --release -p heimdall-build -- rebuild --dry-run

# Rebuild specific countries only
cargo run --release -p heimdall-build -- rebuild --country dk,no

# Force redownload everything
cargo run --release -p heimdall-build -- rebuild --redownload

# Custom config and state file
cargo run --release -p heimdall-build -- rebuild \
  --config data/sources.toml \
  --state-file data/rebuild-state.json
```

## Key design decisions

- **FST for lookup, not search.** The `fst` crate gives us O(key_length) exact lookup and built-in Levenshtein automaton. Not suitable for substring/prefix search (that's what the planned ngram FST is for).

- **Street-grouped address storage.** One record per street segment instead of one per address. House numbers stored as delta-encoded coordinates. 70% compression vs per-address storage.

- **Nearest-centroid admin assignment.** Not proper point-in-polygon — uses the nearest county/municipality centroid. ~95% accurate, fails near borders (Lund→Staffanstorp, etc). Acceptable for PoC.

- **NodeCache trait for planet scaling.** `MmapNodeCache` (default) uses a sparse mmap'd temp file — O(1) direct-indexed by node ID, ~500MB RAM for Germany. `SortedVecNodeCache` is the in-memory alternative (16 bytes/node) but uses ~7GB for Germany.

- **Per-country normalizer configs.** All in `data/normalizers/`. `sv.toml` for Swedish, `no.toml` for Norwegian, `de.toml` for German, etc. Known_variants handle diacritic-free spellings (Tromso→Tromsø, Munich→München) and historical names (Christiania→Oslo). German config adds `[diacritics]` (ä→ae, ö→oe, ü→ue), `[char_equivalences]` (ß↔ss), and `[phonetic] engine = "cologne"` for Kölner Phonetik. The normalizer TOML is auto-copied as `sv.toml` into each index directory at build time.

- **Configurable phonetic encoding.** TOML `[phonetic] engine` selects the encoder: `"cologne"` for Kölner Phonetik (German), default Swedish Metaphone for Nordic languages. Each language family gets the right encoder — critical for fuzzy matching accuracy.

- **SSR place names as supplemental source.** Kartverket's Sentralt Stedsnavnregister (SSR) provides 784K official Norwegian place names (farms, mountains, lakes, rivers, etc.) parsed from 6.5GB GML via streaming `quick-xml`. Merged with spatial dedup (grid-cell proximity + name match) — 524K new places added to Norway's 497K OSM places. Uses negative `stedsnummer` as `osm_id` to avoid collision. Configured via `[places_source]` in `sources.toml`, runs as step 2b in the rebuild pipeline.

- **Automated rebuild.** `heimdall-build rebuild` reads `data/sources.toml` for country/source definitions. Change detection uses cheap HTTP checks (Geofabrik state.txt sequence numbers, Photon MD5 hashes, DAWA sequence numbers, Kartverket ETags) before downloading. Countries are built sequentially with mmap node cache (~1-2GB RAM), parallel pack steps, and auto-cleanup of downloads. State persisted to `data/rebuild-state.json` after each country for crash safety. `heimdall-build download` pre-fetches all sources (up to 4 concurrent) without building — use with `rebuild --skip-download` for a two-step workflow (download all, then build sequentially).

## Rebuild pipeline

### Per-country pipeline

```
1. Extract OSM        → extract::extract_places()        (skip if PBF unchanged + parquet exists)
2. Merge national     → dawa/geonorge/dvv + merge        (addresses)
2b. Merge places_src  → ssr::read_ssr_places() + merge   (places — Norway SSR)
3. Merge Photon       → lucene::read_all_json() + photon::parse_es_documents() + merge
4. Enrich admin       → enrich::enrich()
5. Pack places        → pack::pack()                     ┐ parallel in standard/fast mode
6. Pack addresses     → pack_addr::pack_addresses()      ┘
7. Write meta.json
```

Photon-only countries (GB) skip step 1-2 and use Photon data as the primary source.
Step 2b only runs for countries with `[places_source]` in sources.toml (currently Norway/SSR).

### Change detection (no download needed)

| Source | Method | Cost |
|--------|--------|------|
| Geofabrik PBF | GET `state.txt`, compare `sequenceNumber` | ~100 bytes |
| Photon | GET `.md5` file, compare hash | ~50 bytes |
| DAWA | GET `/replikering/senestesekvensnummer` JSON | ~50 bytes |
| Kartverket | HEAD on ZIP, compare `ETag` | 1 HEAD request |
| DVV | Always rebuild (API, no version endpoint) | Free |
| LINZ | Always rebuild (manual download, API token required) | Free |

### Config and state files

- **`data/sources.toml`** — Checked-in config with country definitions, source URLs, RAM budgets. Edit this to add/remove countries or update URLs.
- **`data/rebuild-state.json`** — Gitignored. Tracks ETags, sequence numbers, MD5 hashes, and local download paths. Saved after each wave for crash recovery.
- **`data/rebuild-report-{timestamp}.log`** — Gitignored. Per-step timing, RSS memory snapshots, and summary stats.

## Index directory layout

```
data/index-se/
  records.bin          Place records (24 bytes each) + string pool
  fst_exact.fst        Normalized name → record_id
  fst_phonetic.fst     Swedish metaphone → record_id
  fst_ngram.fst        (placeholder, empty)
  addr_streets.bin     Street-grouped address store
  fst_addr.fst         normalized_street:municipality_id → street_id
  geohash_index.bin    Spatial index for reverse geocoding
  admin.bin            County/municipality hierarchy (bincode Vec<AdminEntry>)
  admin_map.bin        OSM ID → (admin1_id, admin2_id) mapping (build-time only)
  places.parquet       Intermediate extraction output (build-time only)
  addresses.parquet    Intermediate address output (build-time only)
  sv.toml              Normalizer config (auto-copied from data/normalizers/)
  meta.json            Build metadata
```

## Adding a new country

1. Create `data/normalizers/{cc}.toml` with abbreviations, stopwords, known_variants
2. Update `detect_country()` in `extract.rs` with the country's bounding box
3. Update `enrich.rs` admin filter if needed (the `is_in_nordic` bbox)
4. Add the country to `data/sources.toml` with OSM URL, Photon URL, and optional national source
5. Run `heimdall-build rebuild --country {cc}` (or manual: `build --input {pbf} --output data/index-{country}`)
6. The normalizer TOML is auto-copied as `sv.toml` into the index directory
7. Add `--index data/index-{country}` to the server command
8. Country detection in `load_country_index()` in the API uses directory name patterns

## Known issues

- **Christiania → Oslo** known_variant not resolving. The normalizer produces "oslo" as first candidate but the query pipeline sometimes skips it. Needs debugging in the search handler's candidate iteration.

- **Nearest-centroid admin** misassigns places near county/municipality borders. Proper fix: polygon containment using the relation geometry we already extract. Medium effort.

- **Address FST key collision.** Municipality IDs assigned by nearest-centroid differ between places and addresses for the same location. Fixed by FST range scan + city coordinate distance filtering, but adds latency for ambiguous streets.

- **Norwegian admin count** (414) includes historical kommuner from pre-2024 mergers. Should filter to current 356 kommuner only.

## Data sources

| Country | Places | Addresses | Auth needed |
|---------|--------|-----------|-------------|
| Sweden | OSM PBF (Geofabrik) | OSM addr:* tags (903K) | None |
| Sweden | — | Lantmäteriet Belägenhetsadress (5M) | Legal review required |
| Norway | OSM PBF (Geofabrik) | OSM addr:* tags (2.6M) | None |
| Norway | SSR place names (Kartverket, 310MB ZIP / 6.5GB GML, 784K places) | — | None (downloaded + merged) |
| Norway | — | Kartverket Matrikkelen (2.6M CSV) | None (downloaded + merged) |
| Denmark | OSM PBF (Geofabrik) | DAWA API (~2.5M, no auth) | None |
| Finland | OSM PBF (Geofabrik) | DVV via OGC API (3.7M) | None |
| Germany | OSM PBF (Geofabrik, ~4GB) | OSM addr:* tags (~20M) | None |
| GB | Photon extract (Graphhopper) | Photon (905K places, 4.6M addr) | None |
| United States | OSM PBF (Geofabrik) + USGS GNIS Domestic Names (~1M+ places, public domain) | TIGER/Line 2025 + OpenAddresses | None |
| United States | TIGER/Line 2025 COUSUB (towns/townships, strong-MCD states only) | — | None (downloaded by tiger-import) |
| United States | TIGER/Line 2025 AIANNH (federally-recognised tribal areas) | — | None (downloaded by tiger-import) |
| United States | Census Gazetteer 2024 counties (county populations, backfill) | — | None (downloaded by tiger-import) |
| United States | simplemaps US ZIPs CSV (CC BY 4.0) — HUD-equivalent ZIP→city crosswalk | — | None (downloaded by tiger-import) |
| Australia | OSM PBF (Geofabrik) | G-NAF (15.9M, CC BY 4.0) | None |
| Canada | OSM PBF (Geofabrik) | NAR (15.8M, StatCan Open Licence) | None |
| New Zealand | OSM PBF (Geofabrik) | LINZ NZ Addresses (2.3M, CC BY 4.0) | Free LINZ account + API token |
| Netherlands | OSM PBF (Geofabrik) | BAG via NLExtract (9.5M, CC0) | None |
| France | OSM PBF (Geofabrik) | BAN per-département (~26M, Etalab OL) | None (auto-download) |
| Switzerland | OSM PBF (Geofabrik) | swisstopo Gebäudeadressen (2.2M, OGD) | None |
| Austria | OSM PBF (Geofabrik) | BEV Adressregister (2.8M, BEV PSI) | None |
| Belgium | OSM PBF (Geofabrik) | BeST OpenAddresses (4.5M, CC BY 4.0) | None |
| Czech Republic | OSM PBF (Geofabrik) | RÚIAN (2.9M, CC BY 4.0) | None |
| Poland | OSM PBF (Geofabrik) | PRG address points (8M, free) | None |
| Estonia | OSM PBF (Geofabrik) | ADS (400K) | None |
| Latvia | OSM PBF (Geofabrik) | VZD (600K, CC BY) | None |
| Lithuania | OSM PBF (Geofabrik) | govlt SQLite (700K, free) | None |
| Japan | OSM PBF (Geofabrik) | ABR via abr-geocoder SQLite (~50M, PDL 1.0) | Run abr-geocoder first |
| South Korea | OSM PBF (Geofabrik) | juso.go.kr road-name addresses (~6M) | Free registration |
| Brazil | OSM PBF (Geofabrik) | CNEFE 2022 (~107M, open) | None |
| Kiribati, Nauru, Niue, Palau, Tuvalu | OSM PBF (Geofabrik) | OSM addr:* tags | None |
| 163 others | Photon extract (Graphhopper) | Photon (ODbL) | None |

---
> Source: [martinarnell/heimdall](https://github.com/martinarnell/heimdall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
