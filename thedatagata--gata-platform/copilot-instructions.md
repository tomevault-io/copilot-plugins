## gata-platform

> Multi-tenant analytics platform. Customers (tenants) connect their data sources (ad platforms, ecommerce, analytics tools). We land their raw data, push it through a schema-enforced pipeline, and produce tenant-isolated star schemas for reporting.

# GATA Platform — Claude Code Context

## What This Is
Multi-tenant analytics platform. Customers (tenants) connect their data sources (ad platforms, ecommerce, analytics tools). We land their raw data, push it through a schema-enforced pipeline, and produce tenant-isolated star schemas for reporting.

## Architecture (5 Layers)

```
sources/{tenant}/{platform}/     → Raw data landing (auto-generated)
staging/{tenant}/{platform}/     → Schema hash routing + push to master models (auto-generated)
platform/master_models/          → Multi-tenant tables with raw_data_payload JSON column
intermediate/{tenant}/{platform}/ → Extract typed fields from JSON, tenant-filtered views
analytics/{tenant}/              → Star schema (facts + dims) via engine/factory macros
```

**Layers 1-3 are auto-generated** by `scripts/onboard_tenant.py`. Staging models use `sync_to_master_hub()` post-hook to MERGE data into master model tables.

**Layers 4-5 are hand-written.** Intermediate models extract fields from `raw_data_payload` JSON. Analytics models call factory macros that call engine macros.

## Key Patterns

### Engine/Factory Pattern (Shell Architecture)
- **Engines** (`macros/engines/{domain}/`): Source-specific SQL that reads from intermediate models and outputs a canonical column set. Example: `engine_facebook_ads_performance(tenant_slug)` reads `int_{tenant}__facebook_ads_facebook_insights` and outputs `tenant_slug, source_platform, report_date, campaign_id, ad_group_id, ad_id, spend, impressions, clicks, conversions`.
- **Factories** (`macros/factories/`): Config-driven engine discovery. Call `get_tenant_config(tenant_slug)` to find enabled sources, then dynamically resolve engine macros via `context.get('engine_' ~ source ~ '_performance')`. Example: `build_fct_ad_performance('tyrell_corp')` discovers facebook_ads, google_ads, instagram_ads from config and UNION ALLs their engine outputs.
- **Shell Models** (`models/analytics/{tenant}/`): Single-line SQL files that call a factory with only the tenant slug. Example: `{{ build_fct_ad_performance('tyrell_corp') }}`

### JSON Extraction (DuckDB Syntax)
All master models store raw data in a `raw_data_payload` JSON column. Intermediate models extract typed fields:
```sql
CAST(raw_data_payload->>'$.field_name' AS BIGINT)     -- scalar to type
raw_data_payload->>'$.field_name'                       -- scalar to VARCHAR
raw_data_payload->'$.nested_object'                     -- keep as JSON
```

### Push Circuit (sync_to_master_hub)
Staging models are views that SELECT from sources. A `post_hook` in `generate_staging_pusher` calls `sync_to_master_hub(master_model_id)` which does a MERGE INTO the master model table USING the staging view. The post-hook ensures the view exists before the MERGE fires. Master model tables must exist before staging models can push to them.

### Master Model Incremental Pattern
Master models use `materialized='incremental'` with `full_refresh=false` to retain historical data across runs. The mechanism:
1. **First run** → dbt creates the table (SELECT ... WHERE 1=0 produces the empty schema)
2. **Staging post-hooks** → `sync_to_master_hub()` MERGEs data in (deduped by tenant + platform + payload hash)
3. **Subsequent runs** → dbt sees table exists, runs incremental append (0 new rows), existing data untouched
4. **Staging post-hooks** → MERGE inserts only genuinely new rows

**`full_refresh=false`** means `dbt run --full-refresh` will NOT drop master models. This is intentional — master models are append-only sinks. To truly reset one, DROP the table manually in the warehouse.

**Safe refresh commands:**
```bash
# Refresh reporting layer only (most common)
dbt run --selector reporting_refresh

# Full refresh everything EXCEPT master models
dbt run --full-refresh --selector safe_full_refresh

# Inspect master models (rarely needed)
dbt run --selector master_models_only
```

### Staging Pusher (generate_staging_pusher)
The macro in `macros/onboarding/generate_staging_pusher.sql` creates staging views that:
1. SELECT from tenant source tables
2. Pack all source columns into a single `raw_data_payload` JSON column via `row_to_json(base)`
3. Fire `sync_to_master_hub()` as a `post_hook` (runs after view creation)

## Active Tenants

| Tenant | Ad Sources | Ecommerce | Analytics | Status |
|--------|-----------|-----------|-----------|--------|
| tyrell_corp | facebook_ads, instagram_ads, google_ads | shopify | google_analytics | Active |
| wayne_enterprises | bing_ads, google_ads | bigcommerce | google_analytics | Active |
| stark_industries | facebook_ads, instagram_ads | woocommerce | mixpanel | Active |

## Important File Locations

| What | Where |
|------|-------|
| dbt project root | `warehouse/gata_transformation/` |
| Tenant config | `tenants.yaml` (project root) |
| Connector catalog | `supported_connectors.yaml` (project root) |
| Engine macros | `warehouse/gata_transformation/macros/engines/{analytics,ecommerce,paid_ads}/` |
| Factory macros | `warehouse/gata_transformation/macros/factories/` |
| Intermediate unpacker macro | `warehouse/gata_transformation/macros/factories/generate_intermediate_unpacker.sql` |
| Push macro | `warehouse/gata_transformation/macros/onboarding/sync_to_master_hub.sql` |
| Staging pusher macro | `warehouse/gata_transformation/macros/onboarding/generate_staging_pusher.sql` |
| Intermediate models | `warehouse/gata_transformation/models/intermediate/{tenant}/` |
| Analytics models | `warehouse/gata_transformation/models/analytics/{tenant}/` |
| Master models | `warehouse/gata_transformation/models/platform/master_models/` |
| Platform governance | `warehouse/gata_transformation/models/platform/{hubs,satellites,ops}/` |
| Mock data generators | `services/mock-data-engine/sources/{domain}/{platform}/` |
| Onboarding scripts | `scripts/onboard_tenant.py`, `scripts/initialize_connector_library.py` |
| BSL semantic configs | `services/platform-api/semantic_configs/{tenant_slug}.yaml` |
| Platform API | `services/platform-api/main.py` |

## dbt Commands

**Always run dbt using:** `uv run --env-file ../../.env dbt <command> --target <target>`
**Always run from:** `warehouse/gata_transformation/`

Example: `uv run --env-file ../../.env dbt run --target dev`

## dbt Targets

- **dev**: MotherDuck (`md:my_db`). Requires `MOTHERDUCK_TOKEN` env var in `.env` file at project root.
- **sandbox**: Local DuckDB file (`warehouse/sandbox.duckdb`). Uses `threads: 1` to avoid file locking. Fully functional — no MotherDuck dependency needed.

## Current Database State (Feb 2026)

**Full pipeline operational on both targets.** Last successful run: 133 PASS, 0 ERROR, 0 SKIP.

### Model Counts
- **Total models:** 136 (132 materialized + 4 hooks)
- **Intermediate:** 20 models (8 tyrell_corp, 6 wayne_enterprises, 6 stark_industries)
- **Analytics:** 18 models (6 per tenant: fct_ad_performance, fct_orders, fct_sessions, fct_events, dim_campaigns, dim_users)
- **Boring Semantic Layer:** 1 model cataloging all star schema tables with column metadata

### MotherDuck (dev target)
- Raw data in tenant-specific schemas: `tyrell_corp.*`, `wayne_enterprises.*`, `stark_industries.*`
- Master model tables populated via staging MERGE (data flowing end-to-end)
- All intermediate tables and analytics tables materialized with data
- `_sources.yml` files include `database: "my_db"` for MotherDuck resolution

### Sandbox (local target)
- Raw data in tenant-specific schemas within `warehouse/sandbox.duckdb`
- Full parity with dev — same 133 models pass
- `_sources.yml` files omit `database` key (local DuckDB resolves without it)
- Data landed via `dlt.destinations.duckdb(credentials=sandbox_path)` in orchestrator

### Execution Order Note
With `materialized='incremental'` and `full_refresh=false`, master models retain data across runs. However, on a **first-ever run** (or after a manual table drop), master models start empty and staging MERGE post-hooks populate them. Intermediate models may execute before staging post-hooks fire, resulting in empty intermediate/analytics tables on the first pass. **Fix:** Use the reporting_refresh selector for a second pass:
```bash
uv run --env-file ../../.env dbt run --target <target> --selector reporting_refresh
```

## Onboarding Workflow (sandbox)

```bash
# 1. Initialize connector library (registers schema hashes → master model mappings)
python scripts/initialize_connector_library.py sandbox

# 2. Set tenant status to "onboarding" in tenants.yaml, then for each tenant:
python scripts/onboard_tenant.py <tenant_slug> --target sandbox --days 30

# 3. Run full dbt pipeline
cd warehouse/gata_transformation
uv run --env-file ../../.env dbt run --target sandbox
```

For dev (MotherDuck), replace `sandbox` with `dev` in all commands.

## Reporting Layer (Intermediate + Analytics)

### Intermediate Unpacker Macro (`generate_intermediate_unpacker`)
Reusable macro that generates intermediate models from master models. Parameters:
- `tenant_slug`, `source_platform`, `master_model_id`, `columns` (list of `{json_key, alias, cast_to}` dicts)
- Optional: `json_op: '->'` to keep as JSON, `expression` for raw SQL overrides (e.g., computed columns)
- Always includes passthrough columns: `tenant_slug`, `source_platform`, `tenant_skey`, `loaded_at`, `raw_data_payload`

Models with computed columns (Google Ads `cost_micros / 1000000.0`, Mixpanel `prop_time * 1000`, Shopify/WooCommerce JSON line_items) are written as raw SQL instead of using the macro.

### Star Schema Tables (per tenant)
| Model | Description |
|-------|-------------|
| `fct_{slug}__ad_performance` | UNION ALL of ad engines (spend, impressions, clicks, conversions by ad_id/date) |
| `fct_{slug}__orders` | Ecommerce orders (order_id, total_price, currency, customer info) |
| `fct_{slug}__sessions` | Sessionized analytics events (30-min window, attribution, conversion flags) |
| `fct_{slug}__events` | Raw analytics event stream with attribution and optional ecommerce data |
| `dim_{slug}__campaigns` | Campaign dimension (campaign_id, name, status across ad platforms) |
| `dim_{slug}__users` | Unified user dimension with identity resolution across analytics + ecommerce |

### Boring Semantic Layer (`platform_ops__boring_semantic_layer`)
Auto-catalogs all `fct_*` and `dim_*` tables using `INFORMATION_SCHEMA`. Outputs one row per star schema table with: `tenant_slug`, `table_type` (fact/dimension), `subject`, `table_name`, `semantic_manifest` (JSON array of column metadata).

### BSL Semantic Configs
Per-tenant YAML config files at `services/platform-api/semantic_configs/{tenant_slug}.yaml` provide semantic context on top of the raw BSL metadata. Each config defines:
- **dimensions**: Columns for grouping/filtering with types (string, date, boolean, timestamp_epoch)
- **measures**: Columns for aggregation with default `agg` (sum, avg, count_distinct)
- **calculated_measures**: Derived metrics as SQL expressions (CTR, CPC, CPM, AOV, conversion_rate)
- **joins**: How fact tables relate to dimension tables (column mappings + join type)

Served via `GET /semantic-layer/{tenant_slug}/config` endpoint in the platform API.

## Platform Governance Models

These track tenant config changes at table-level granularity (tenant + source + table):

- **`platform_sat__tenant_config_history`**: Unpacks `sources_config` JSON from tenant manifest using `json_keys()` + `from_json()`. Outputs: `tenant_slug, tenant_skey, source_name, table_name, table_logic, logic_hash, updated_at`
- **`hub_key_registry`**: Generates surrogate keys from config history for change tracking
- **`platform_sat__tenant_source_configs`**: Latest config per tenant/source/table (thin wrapper on config_history)
- **`platform_ops__source_table_candidate_registry`**: Joins config history with physical table inventory for onboarding recommendations

## Resolved Issues (Feb 2026)

1. **Staging MERGE bootstrap — FIXED:** Moved `sync_to_master_hub()` from inline `{% do %}` (ran before view creation) to `post_hook` in config (runs after view creation).
2. **`struct_pack(*)` error — FIXED:** DuckDB can't resolve `*` inside `struct_pack()`. Replaced with `row_to_json(base)`.
3. **`platform_sat__tenant_config_history` JSON unpacking — FIXED:** `json_transform` with `'["JSON"]'` lost object keys. Replaced with `json_keys()` + `from_json()`.
4. **Sandbox target not working — FIXED:** Scripts now route to `warehouse/sandbox.duckdb` when target is `sandbox`. Orchestrator uses `dlt.destinations.duckdb(credentials=path)` to land data in the correct file.
5. **`tojson` quoting bug — FIXED:** Analytics engines were using `tojson` which produces double quotes. Fixed to use single-quote wrapping via `{% for %}` loop.
6. **Windows cp1252 emoji crashes — FIXED:** Replaced emoji characters in Python scripts with ASCII-safe `[TAG]` prefixes.
7. **Master model historical data loss — FIXED:** Changed master models from `materialized='table'` to `materialized='incremental'` with `full_refresh=false`. Added `selectors.yml` with `safe_full_refresh` and `reporting_refresh` selectors. Master models now retain data across runs; `--full-refresh` is blocked at the model level.

## Conventions

- Intermediate models: `int_{tenant_slug}__{platform}_{object}.sql`, materialized as tables
- Analytics models: `fct_{tenant_slug}__{metric}.sql` or `dim_{tenant_slug}__{dimension}.sql`, materialized as tables
- Staging models: `stg_{tenant_slug}__{platform}_{object}.sql`, materialized as views with `sync_to_master_hub()` post-hook
- Master models: `platform_mm__{connector_api_version}_{object}.sql`
- All intermediate models must include `tenant_slug`, `source_platform`, `tenant_skey`, `loaded_at` columns plus `raw_data_payload` as the last column
- Python scripts must use ASCII-safe print statements (no emojis) for Windows compatibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thedatagata) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
