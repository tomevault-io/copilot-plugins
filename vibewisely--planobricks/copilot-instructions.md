## planobricks

> Planogram compliance dashboard for Retail/CPG, built on Databricks. Uses AI vision

# PlanoBricks

Planogram compliance dashboard for Retail/CPG, built on Databricks. Uses AI vision
(Foundation Model API + Claude Haiku 4.5) to detect shelf products and classify brands,
then verifies compliance against schematic planogram references using Needleman-Wunsch
sequence alignment. Multi-store capable with AI-powered schematic creation from images.

**Full product specification**: See `PRD.md` for the comprehensive Product Requirements Document.

## Product Domain

- **Planogram**: Expected shelf layout defining which brands go where, organized by shelf rows
- **Schematic**: Reference planogram built from multi-image consensus (auto) or user-created (custom)
- **Realogram**: Actual shelf layout captured via photograph, compared against the schematic
- **Compliance score**: Ratio of correctly placed products to expected (0.0–1.0) via NW alignment
- **Store**: A location grouping for shelf images and schematics (e.g., Store A, Store B)
- **Brand identification pipeline**: `ai_query('databricks-claude-haiku-4-5', ...)` on 13K+ product crops
- **Unity Catalog**: `serverless_stable_wunnava_catalog` catalog with `planobricks_reference` schema
- **SchematicKey**: Tuple of `(planogram_id, num_shelves, shelf_rank)` — e.g., `P01/3s/R1`

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Databricks App (Dash 4.0 + dash-bootstrap-components)      │
│  ┌──────────┐ ┌──────────────┐ ┌──────────┐ ┌───────────┐  │
│  │ Overview  │ │  Inspector   │ │  Editor  │ │  Dataset  │  │
│  │ (KPIs,   │ │  (Photo +    │ │(Schematic│ │  (Info,   │  │
│  │  charts, │ │  bounding    │ │ CRUD +   │ │  stats)   │  │
│  │  table)  │ │  boxes +     │ │ AI image │ │           │  │
│  │          │ │  schematic + │ │ upload)  │ │           │  │
│  │          │ │  crop/commit)│ │          │ │           │  │
│  └──────────┘ └──────────────┘ └──────────┘ └───────────┘  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Store Selector (navbar) — multi-store switching      │   │
│  └──────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│  grocery_data.py   — Data loading, caching, per-store data  │
│  planogram_engine.py — Row clustering, NW alignment, scores │
│  planogram_store.py — Schematic CRUD (local + UC Volume)    │
│  store_manager.py  — Multi-store management & persistence   │
├─────────────────────────────────────────────────────────────┤
│  UC Volumes (images, schematics JSON)                       │
│  Foundation Model API (brand detection from shelf images)    │
│  Databricks Asset Bundles (deployment)                      │
└─────────────────────────────────────────────────────────────┘
```

## Project Structure

```
planobricks/
├── src/app/                    # Dash application (deployed to Databricks Apps)
│   ├── app.py                  # Main Dash app — layout, all callbacks (~1800 lines)
│   ├── grocery_data.py         # Dataset parser, per-store caching, compliance aggregation
│   ├── planogram_engine.py     # NW alignment, schematic builder, compliance computation
│   ├── planogram_store.py      # Schematic JSON persistence (local + UC Volume, per-store)
│   ├── store_manager.py        # Multi-store CRUD and persistence
│   ├── app.yaml                # Databricks App runtime config (env vars)
│   ├── requirements.txt        # App dependencies (dash-bootstrap, plotly, databricks-sdk)
│   └── data/                   # Bundled annotation data
│       ├── annotation.txt      # 354 shelf images × product bounding boxes
│       ├── enriched_products.csv  # AI-identified brand per product (13K rows)
│       └── schematics.json     # Auto + custom schematic planograms
├── notebooks/
│   └── 01_brand_identification.py  # Brand classification via ai_query + Claude Haiku
├── resources/
│   └── planobricks_app.yml     # DABs app resource definition
├── scripts/                    # Test/verification scripts
├── tests/                      # Test suite
├── databricks.yml              # DABs bundle config (dev target, workspace profile)
├── pyproject.toml              # uv/hatch project config, ruff settings
├── PRD.md                      # Full Product Requirements Document
├── CLAUDE.md                   # This file — project context for AI assistants
└── README.md                   # User-facing documentation with screenshots
```

## Key Modules

### `app.py` — Dash Application
- **4 tabs**: Compliance Overview, Shelf Inspector, Schematic Editor, Dataset
- **Store selector** in navbar with create-store modal
- **Shelf Inspector**: actual photo + bounding box overlay + schematic reference + crop slider
  for re-evaluating compliance on a column sub-range (commit/reset workflow)
- **Schematic Editor**: CRUD (new, clone, delete, reset-to-auto), row editing via pipe-separated
  brand lists, AI image upload for automated brand detection
- **AI Brand Detection**: uploads shelf image → calls Foundation Model API
  (`databricks-claude-haiku-4-5`) via `w.api_client.do()` REST call → parses response into
  schematic rows → saves as custom schematic
- All tab content rendered in the initial layout (not dynamically) so Dash's client-side
  callback resolver can wire up all component IDs at page load

### `planogram_engine.py` — Core Algorithms
- `cluster_into_rows()` — Groups products into shelf rows by y-coordinate gaps
- `needleman_wunsch()` — Global sequence alignment (match=+2, mismatch=-1, gap=-2)
- `build_schematics()` — Multi-image consensus per (planogram_id, num_shelves, shelf_rank)
- `compute_compliance()` — Row-by-row NW alignment, returns `ShelfComplianceResult`
- Key types: `SchematicKey`, `SchematicPlanogram`, `SchematicRow`, `AlignedPair`

### `planogram_store.py` — Schematic Persistence
- JSON storage: local file (`data/schematics.json`) + UC Volume sync
- Per-store schematics: `data/store_schematics/{store_id}.json`
- Preserves custom edits across auto-regeneration (`origin: "auto" | "custom"`)

### `grocery_data.py` — Data Layer
- Parses `annotation.txt` (354 shelf images, 13K+ products)
- Merges `enriched_products.csv` for AI-identified brands
- Per-store caching: Store A uses bundled dataset; other stores start empty
- `get_data(store_id)` returns all shelves, schematics, compliance results

### `store_manager.py` — Multi-Store Management
- Default "Store A" with bundled Grocery Dataset
- Users create additional stores (Store B, etc.) with name + description
- Persisted as `data/stores.json` + UC Volume sync

## Development

- **Package manager**: `uv` — use `uv add <pkg>` to add dependencies, `uv run <cmd>` to run scripts
- **Python version**: 3.11+
- **Dash version**: 4.0.0 (important: `allow_duplicate=True` requires `prevent_initial_call=True`)
- **Run locally**: `cd src/app && python app.py` (port 8080)
- **Test**: `uv run pytest`
- **Lint**: `uv run ruff check .`
- **Format**: `uv run ruff format .`

## Deployment

- **Deploy**: `databricks bundle deploy --target dev`
- **App deploy**: `databricks apps deploy planobricks-dev --source-code-path "/Workspace/Users/venkata.wunnava@databricks.com/.bundle/planobricks/dev/files/src/app"`
- **Logs**: `databricks apps logs planobricks-dev`
- **App URL**: `https://planobricks-dev-7474651516019640.aws.databricksapps.com`
- **Profile**: `planobricks-mar2`

## Conventions

- **Always use ai-dev-kit MCP tools** for Databricks operations (SQL, volumes, apps, endpoints,
  UC objects, etc.) instead of CLI commands. The MCP server is configured as
  `project-0-planobricks-databricks` with 70+ tools including `execute_sql`, `upload_to_volume`,
  `query_serving_endpoint`, `create_or_update_app`, `manage_uc_objects`, etc.
- **Use available skills** when relevant — read and follow the SKILL.md file for specialized
  tasks like app deployment, DABs config, dashboards, SDK usage, etc.
- All tab content in the initial `app.layout` (no dynamic rendering) to ensure callbacks work
- `suppress_callback_exceptions=True` for cross-tab component references
- Use `allow_duplicate=True` on outputs shared across callbacks, always with `prevent_initial_call=True`
- Images loaded from UC Volume via SDK (`w.files.download()`) with local file cache fallback
- Print-based logging (e.g., `[AI Detect]`, `[PlanoBricks]`) for `databricks apps logs` debugging
- Store-scoped data: `gd.get_data(store_id)` for per-store caching

## Databricks

- **Workspace profile**: `planobricks-mar2`
- **Catalog**: `serverless_stable_wunnava_catalog`
- **UC Volume**: `/Volumes/serverless_stable_wunnava_catalog/planobricks_reference/inputs/`
  - `images/ShelfImages/` — 354 shelf photos
  - `schematics.json` — Persisted schematic planograms
  - `stores.json` — Store metadata
- **Foundation Model API**: `databricks-claude-haiku-4-5` — used for brand identification
  from product crops and shelf image analysis
- **App auth**: M2M OAuth (`auth_type=oauth-m2m`) — SDK auto-configured in Databricks Apps
- **DABs target**: `dev` (development mode)

## Known Constraints

- UC Volume write requires `WRITE VOLUME` privilege on the app service principal
- Shelf images only bundled for Store A (Grocery Dataset); other stores are schematic-only
- AI brand detection falls back to "Unknown" placeholders if FMAPI endpoint is unreachable
- Dash 4.0 quirk: large multi-output callbacks (18+ outputs) can trigger `IndexError` in
  `_prepare_grouping`; split into smaller callbacks as needed

---
> Source: [vibewisely/planobricks](https://github.com/vibewisely/planobricks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
