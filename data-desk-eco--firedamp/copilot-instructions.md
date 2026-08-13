## firedamp

> Methane plume aggregator with per-plume AI source attribution.

# Firedamp

Methane plume aggregator with per-plume AI source attribution.

## Architecture

A [cartograph](~/Tools/cartograph) app — the map system itself is documented
there — with all firedamp behaviour in `web/config.js` plus two hook
modules. The plume aggregation ETL lives in the sibling **etl** repo
(`~/Tools/etl`, 6-hourly publish to the archive); the offline research agent —
the sibling **ch4id** repo — produces the attributions. Served on GitHub
Pages; there is no backend service.

- `web/config.js` — the declarative cartograph config: sources (plume parquet →
  geojson, clustered to z4), four per-source dd flare-marking symbol layers
  (dd map palette: cm cyan, imeo magenta, sron yellow, ghgsat orange, dd green; one
  fixed size, t/hr labels) plus a non-interactive white cluster layer
  labelled with summed t/hr,
  attribution/date filters, key (toggleable rate ranges + source toggles),
  data table tabs (detections, attributions), detail panel.
- `web/attribution.js` — attribution lookup (full attributions.parquet into a
  Map at boot, keyed by plume display id) + rendering (source label linking to
  osm.org or flying to the feature, "(confidence: …)" after the label,
  paragraph, evidence links), and the per-plume daily-mean surface wind stat
  (Open-Meteo archive).
- `web/candidates.js` — candidate sources from the provider-owned MapStand,
  OGIM, OSM, and GEM `infrastructure/` tables. DuckDB-Wasm applies bounding-box
  filters: viewport sweep past z13 (padded rect)
  + per-plume radius query on selection (3 km, 10 km for coarse sensors;
  nearest 300; attributed ids always kept, rect stretched to the assessed
  source point). dd waypoint markings, orange + larger when attributed, over an
  invisible fat hit layer carrying the hover/click popup. `normId` maps old
  `OSM:way/<id>` attribution ids to ch4id's short `OSM:w<id>` form.
- `web/licences.js` — MapStand oil and gas licence areas in a private,
  Hilbert-sorted GeoParquet file. It applies a viewport sweep past z6,
  purple boundary over a faint wash, beneath every plume layer, with a key
  toggle. **Private deploy only** — the acreage is licensed data, so the layer
  is gated on `PRIVATE` in `config.js`. Known source defect: 55.7% of the
  polygons are rectangles collapsed to
  triangles, because the WMS sweep lets GeoServer generalise geometry to its
  ~2 km render resolution. Position and extent are right, boundaries are not.
- `web/vendor/` — committed: MapLibre, DuckDB-Wasm, Inter, `dd/`
  (design dist) and `cartograph/` (the generic core). Refresh with `make
  vendor` (calls cartograph's vendor.sh, which pulls dd from ~/Tools/design).

Everything generic — dd map shell, panels, key, filters, detail/overlap-nav/
permalinks, search, parquet data layer — lives in vendored cartograph; keep
firedamp specifics out of it (change cartograph and re-vendor instead).

## Plume sources

- **Carbon Mapper** — satellite + aircraft hyperspectral (API → `data/carbon_mapper.csv`)
- **IMEO / MARS** — UNEP methane plume database (public Azure zip → `data/imeo_plumes.csv`)
- **SRON** — TROPOMI weekly plume CSVs (FTP scrape → `data/sron/` → `data/sron_all.csv`)
- **Data Desk (`dd`)** — our own Sentinel-2 retrievals, published by the etl
  repo as the `kind = 'plume'` rows of `data-desk/detections/`. Curation is a
  column now, not a prefix: the table also carries the retrievals the producer
  does not trust, so `config.js` reads `valid = true` itself. Public like every
  other source. Detail-panel title links to the plume preview under
  `data-desk/assets/`
- **Infrastructure tables** — OGIM, OSM, MapStand and GEM, one
  `<provider>/infrastructure/data.parquet` each, ~15M features in total

## Central data archive

Firedamp serves its data from the shared datadesk CloudFerro bucket
(`https://s3.WAW3-2.cloudferro.com/data-desk-archive`, the `data-bucket` meta;
defined in `~/Tools/data-desk/infra/archive.sh`; layout in `data-desk/docs/archive/`).
The ETL repository publishes the provider `detections/` and `infrastructure/`
tables. Firedamp reads four fixed `<provider>/detections/data.parquet` objects
filtered to `kind = 'plume'`, and queries the infrastructure tables directly.

## Frontend data flow

`config.js` names the four public provider detections objects and reads them
through DuckDB-Wasm. Every plume whose id appears in
`attributions.parquet` gets `attr: 1`, which feeds the Attribution filter.
The detail panel is cartograph-generic (title link to CM/SRON, coords,
overlap nav across co-located plumes, `#plume=<id>` permalinks alongside
`#map=`); firedamp's `config.js` detail template adds source/sector badges and a stats grid
(rate, wind, satellite, date), and `onShow` fills in wind, the attribution
record and the candidate-source selection. Archive ids carry a provider
namespace (`IMEO:<uuid>`, `SRON:<date>:<lat>:<lon>`, `DD:…`; Carbon Mapper's
are bare), so `config.js` gives the panel a `resolve` that matches a permalink
on the namespace-free spelling — links minted before the archive move still
open their plume.

## Development

```
make vendor        # vendor Cartograph, DuckDB-Wasm, MapLibre, design assets, and Inter
make serve         # dev server on :8000
```

Dev (`localhost`) reads plumes locally if `web/data/plumes.parquet` exists
(copy one from the etl repo), else the archive; infrastructure from the archive. Verify with the `browse` cli: `window.cartograph` exposes `{ map, sources, … }`.

## Deployment

- **Pages**: push to `main` runs `deploy.yml` — `scripts/dist.sh` (copies
  `web/*`, cache-busts entry points / app-local imports / parquet fetches with
  the git SHA — vendor modules stay unbusted so each resolves to one URL = one
  module instance) and deploys. No plume data is in the artifact: the site
  reads the provider `detections/` objects live from the archive.
- **Plumes refresh**: `etl/plumes.yml` every 6h in the etl repo — no redeploy
  needed, the site reads the archive live.
- **Private deploy**: `make deploy-private` — builds `plumes.parquet` via the
  etl repo (including any source held back from the public archive) and bakes
  it into the dist (`dist.sh <sha> local` sets `<meta name="local-plumes">`),
  shipped to Cloudflare Pages project `firedamp-private` behind Cloudflare
  Access; refuses to deploy unless the Access gate is answering. The **only**
  deploy that carries restricted sources — etl's `plumes-upload` refuses to
  publish a parquet containing them. `config.js` lists them in `PRIVATE_SRCS`.
  Data Desk rows are public (curated valid-only).
- **Infrastructure tables**: published by the etl repo
  (re-run when a provider catalogue is rebuilt).

**When adding new web assets**, add them to `scripts/dist.sh` (shared by both
deploy workflows).

---
> Source: [data-desk-eco/firedamp](https://github.com/data-desk-eco/firedamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
