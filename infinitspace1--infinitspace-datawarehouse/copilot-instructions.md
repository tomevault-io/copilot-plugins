## infinitspace-datawarehouse

> > Self-updating protocol: any agent that changes this project must update this file before finishing.

# CLAUDE.md -- InfinitSpace Data Warehouse

> Self-updating protocol: any agent that changes this project must update this file before finishing.

---

## Project Overview

InfinitSpace Data Warehouse is a Python 3.11 Azure Functions ETL project that moves operational data into Azure SQL across these layers:

- `bronze`: raw source payloads
- `silver`: typed and normalized entities
- `ava`: denormalized product availability
- `core`: planned, not implemented

Primary sources today:

- Nexudus API
- Xero API
- CoStar PDF extractor (Real Estate HTTP function)
- Google Maps enrichment utilities exist but are not part of the scheduled Function App

Platform:

- Azure Functions
- Azure SQL
- Azure Blob Storage for raw Nexudus snapshots and Xero invoice PDFs
- Azure Storage Queue for silver fanout

Current deployment docs target:

- resource group: `infinitspace-prod-northeurope-data-rg`
- ETL app: `func-infinitspace-etl`
- storage account: `staccinfinitspaceprod001`

---

## Runtime Topology

Default ETL execution order in UTC:

1. `02:00` `nexudus_to_bronze`
2. `02:30` `bronze_to_silver`
3. queue fanout via `silver_entity_worker`
4. `03:00` `refresh_ava_availability`
5. `04:00` `xero_invoice_sync` (includes PDF caching for invoices missing `pdf_blob_path`)
6. `05:00` `bamboohr_sync`
7. `05:30` `replyio_stats_sync`

Important operational caveat:

- `bronze_to_silver` is schedule-based, not dependency-aware
- `refresh_ava_availability` is also schedule-based
- bronze should finish before silver starts
- silver workers should finish before AVA starts

Flow:

```text
Nexudus API
  -> nexudus_to_bronze
  -> bronze.nexudus_locations
  -> bronze.nexudus_products
  -> bronze.nexudus_contracts
  -> bronze.nexudus_coworker_invoices   (incremental via UpdatedSince watermark)
  -> bronze.nexudus_coworkers           (distinct CoworkerIds from invoices)
  -> bronze.nexudus_resources
  -> bronze.nexudus_extra_services
  -> blob snapshots (nexudus-raw-snapshots container)

bronze_to_silver
  -> Azure Storage Queue: silver-sync-tasks
  -> silver_entity_worker x 7
  -> silver.nexudus_locations
  -> silver.nexudus_location_hours
  -> silver.nexudus_products
  -> silver.nexudus_contracts
  -> silver.nexudus_coworker_invoices
  -> silver.nexudus_coworkers
  -> silver.nexudus_resources
  -> silver.nexudus_extra_services

refresh_ava_availability
  -> EXEC ava.sp_refresh_product_availability
  -> ava.product_availability

Xero API
  -> xero_invoice_sync
  -> bronze.xero_invoices
  -> silver.xero_invoices (pdf_blob_path populated when PDF is cached)
  -> silver.xero_invoice_line_items
  -> silver.xero_tenants
  -> xero.silver_tenants (view alias)
  -> optional bronze.xero_invoice_pdfs

Real Estate (HTTP trigger, optional)
  -> run_costar_extractor (HTTP POST /api/real-estate/costar/run)
  -> costar-extraction-tasks queue
  -> costar_extraction_worker
  -> downloads PDF from Blob (pdf-uploads/<blob_name>)
  -> BuildingContactExtractor (Anthropic API)
  -> uploads XLSX to Blob (excel-outputs/<job_id>_contacts.xlsx)
  -> updates bronze.costar_pdf_extractor_logs (Real Estate DB)

Reply.io API
  -> replyio_stats_sync (timer, 05:30 UTC)
  -> bronze.replyio_sequence_steps
  -> bronze.replyio_sequence_step_performance (daily stats, yesterday)
```

---

## Function App Registration Model

`function_app.py` registers functions based on app settings:

- `ENABLE_ETL_FUNCTIONS=1`
  - registers the production ETL surface
- `ENABLE_ADMIN_FUNCTIONS=1`
  - registers optional admin/debug HTTP routes

Default ETL deployment:

- `ENABLE_ETL_FUNCTIONS=1`
- `ENABLE_ADMIN_FUNCTIONS=0`
- `ENABLE_REAL_ESTATE_FUNCTIONS=0`

Optional admin deployment:

- `ENABLE_ETL_FUNCTIONS=0`
- `ENABLE_ADMIN_FUNCTIONS=1`

Optional Real Estate deployment (can be combined with ETL):

- `ENABLE_ETL_FUNCTIONS=1`
- `ENABLE_REAL_ESTATE_FUNCTIONS=1`
- `ENABLE_ADMIN_FUNCTIONS=0`

This means the default Azure Function App should show only:

- `nexudus_to_bronze`
- `bronze_to_silver`
- `silver_entity_worker`
- `refresh_ava_availability`
- `xero_invoice_sync`
- `replyio_stats_sync`

---

## Repository Structure

```text
Infinitspace-datawarehouse/
  CLAUDE.md
  README.md
  SQL_datawarehouse.md
  function_app.py
  host.json
  requirements.txt
  .env.example
  .funcignore
  functions/
    bronze_nexudus.py
    silver_nexudus.py
    silver_worker.py
    ava_refresh.py
    xero_sync.py
    integrations_admin.py
    admin_health.py
    real_estate_costar.py
    real_estate_costar_worker.py
    replyio_sync.py
  shared/
    azure_clients/
      sql_client.py
      bronze_writer.py
      blob_writer.py
      queue_client.py
      run_tracker.py
      silver_write_locations.py
      silver_writer_products.py
      silver_writer_contracts.py
      silver_writer_resources.py
      silver_writer_extra_services.py
      silver_writer_coworker_invoices.py
      silver_writer_coworkers.py
    nexudus/
      auth.py
      client.py
      transformers/
        locations.py
        products.py
        contracts.py
        resources.py
        extra_services.py
        coworker_invoices.py
        coworkers.py
    bamboohr/
      __init__.py
      client.py
      transformers/
        employees.py
    xero/
      oauth.py
      flow.py
      token_cipher.py
      store.py
      client.py
      invoice_sync.py
      tenant_directory.py
    integrations/
      xero_nexudus_overdue.py
    gmaps/
    replyio/
      __init__.py
      client.py
    real_estate/
      __init__.py
      building_contact_extractor.py   (adapted from AI-REAL-ESTATE repo, uses PyMuPDF)
    azure_clients/
      ...
      costar_queue_client.py
  scripts/
    python_scripts/
      test_local.py
      test_locations_silver.py
      test_products_silver.py
      test_contracts_silver.py
      test_extra_services_silver.py
      inspect_bronze.py
      inspect_product_per_type.py
      enrich_location_gmaps.py
      xero_start_oauth.py
      xero_complete_oauth.py
      xero_get_connections.py
      xero_list_tenants.py
      xero_sync_invoices.py
      xero_list_invoices.py
      xero_download_invoice_pdf.py
      xero_test_contacts.py
      xero_test_invoices.py
      xero_open_auth.py
      xero_exchange_code.py
      xero_refresh_token.py
      xero_register_connection.py
      sync_nexudus_billing.py
      xero_nexudus_link_audit.py
      test_xero_pdf.py
      test_xero_pdf_cache.py
      backfill_xero_pdfs.py
    sql_scripts/
      bronze_layer.sql
      bronze_upsert_constraints.sql
      silver_nexudus_locations_schema.sql
      silver_nexudus_products_schema.sql
      silver_nexudus_contracts_schema.sql
      silver_nexudus_resources_schema.sql
      silver_nexudus_extra_services_schema.sql
      silver_gmaps_locations_schema.sql
      ava_product_availability_schema.sql
      ava_sp_refresh_product_availability.sql
      integrations_nexudus_xero_schema.sql
      nexudus_billing_sync_schema.sql
      xero_invoices_schema.sql
      xero_pdf_blob_migration.sql
      test.sql
  tests/
    test_xero_integration.py
    test_xero_tenant_directory.py
    test_xero_nexudus_invoice_linking.py
  docs/
    deploy.md
    silver_table_relationships.md
  deploy/
    setup_azure_resources.ps1
    setup_azure_resources.sh
```

Legacy Xero helper scripts still exist, but the supported path is now:

- `xero_start_oauth.py`
- `xero_complete_oauth.py`
- DB-backed refresh inside `shared/xero/client.py`

---

## Azure Functions Registry

| Function | File | Trigger | Default schedule or binding | Notes |
|----------|------|---------|-----------------------------|-------|
| `nexudus_to_bronze` | `functions/bronze_nexudus.py` | timer | `0 0 2 * * *` | writes bronze + blob snapshots; coworker invoices/coworkers are incremental via UpdatedSince |
| `bronze_to_silver` | `functions/silver_nexudus.py` | timer | `0 30 2 * * *` | enqueues 7 queue messages (includes coworker_invoices + coworkers) |
| `silver_entity_worker` | `functions/silver_worker.py` | queue | `silver-sync-tasks` | one entity per invocation |
| `refresh_ava_availability` | `functions/ava_refresh.py` | timer | `0 0 3 * * *` | executes AVA stored procedure |
| `xero_invoice_sync` | `functions/xero_sync.py` | timer | `0 0 4 * * *` | syncs all linked Xero tenants + caches PDFs for invoices missing `pdf_blob_path`; reuses the backfill retry/throttle flow |
| admin HTTP routes | `functions/integrations_admin.py` | HTTP | on-demand | only when `ENABLE_ADMIN_FUNCTIONS=1` |
| `test_connections` | `functions/admin_health.py` | HTTP | on-demand | only when `ENABLE_ADMIN_FUNCTIONS=1` |
| `run_costar_extractor` | `functions/real_estate_costar.py` | HTTP POST | `real-estate/costar/run` | only when `ENABLE_REAL_ESTATE_FUNCTIONS=1` — enqueues only, returns 202 |
| `costar_extraction_worker` | `functions/real_estate_costar_worker.py` | queue | `costar-extraction-tasks` | only when `ENABLE_REAL_ESTATE_FUNCTIONS=1` — does the actual extraction |
| `bamboohr_sync` | `functions/bamboohr_sync.py` | timer | `0 0 5 * * *` | syncs all BambooHR employees to bronze + silver; join key: `work_email` |
| `replyio_stats_sync` | `functions/replyio_sync.py` | timer | `0 30 5 * * *` | syncs Reply.io sequence steps + daily step performance stats to bronze |

---

## Data Model Summary

### Real Estate (CoStar extractor)

- uses `bronze.costar_pdf_extractor_logs` (in Real Estate DB, not datawarehouse DB)
- connection string: `AZURE_SQL_PDF_JOBS_CONNECTION_STRING`
- extractor module: `shared/real_estate/building_contact_extractor.py`
  (adapted from AI-REAL-ESTATE repo, uses PyMuPDF — no system dependencies)

### Bronze

- `bronze.nexudus_locations`
- `bronze.nexudus_products`
- `bronze.nexudus_contracts`
- `bronze.nexudus_resources`
- `bronze.nexudus_extra_services`
- `bronze.nexudus_coworker_invoices`
- `bronze.nexudus_coworkers`
- `bronze.xero_invoices`
- `bronze.xero_invoice_pdfs` — stores `blob_path` reference, not raw bytes
- `bronze.bamboohr_employees`
- `bronze.replyio_sequence_steps`
- `bronze.replyio_sequence_step_performance`

Nexudus bronze rows are latest-payload upserts on `source_id`, not append-only history.

### Silver

- `silver.nexudus_locations`
- `silver.nexudus_location_hours`
- `silver.nexudus_products`
- `silver.nexudus_contracts`
- `silver.nexudus_resources`
- `silver.nexudus_extra_services`
- `silver.nexudus_coworker_invoices`
- `silver.nexudus_coworkers`
- `silver.xero_invoices` — includes `pdf_blob_path`, `pdf_cached_at`
- `silver.xero_invoice_line_items`
- `silver.xero_tenants`
- `silver.location_nearby_pois`
- `silver.location_transit_stations`
- `silver.location_neighborhoods`
- `silver.xero_overdue_invoice_contacts` — view joining overdue Xero invoices to Nexudus customer email data
- `silver.bamboohr_employees` — join key: `work_email` → `silver.nexudus_coworkers.email`

### AVA

- `ava.product_availability`
  - rebuilt daily
  - populated by stored procedure
  - no incremental logic

### Meta

- `meta.sync_runs`
- `meta.sync_errors`
- `meta.gmaps_enrichment_log`
- `meta.xero_oauth_states`
- `meta.xero_connections`
- `meta.xero_tenants`

### Xero Directory

- canonical table: `silver.xero_tenants`
- SQL view alias: `xero.silver_tenants`
- one row per Xero tenant for a connection
- location columns are copied from the best matched `silver.nexudus_locations` row
- `community_manager_name` is intentionally a placeholder for now and is preserved on refresh

---

## Key Technical Behaviors

### Nexudus Bronze

- `functions/bronze_nexudus.py`
- fetch order:
  - locations
  - products
  - contracts
  - coworker_invoices (incremental — passes `UpdatedSince` watermark from `meta.sync_runs` on subsequent runs)
  - coworkers (fetches only distinct CoworkerIds seen in coworker_invoices)
  - resources
  - extra_services
- each entity writes a `RunTracker` row
- each entity also writes a blob snapshot

### Silver Fanout

- `functions/silver_nexudus.py` only enqueues work
- `functions/silver_worker.py` performs the actual transformation
- queue retries are safe because silver writes are idempotent upserts
- poison queue: `silver-sync-tasks-poison`
- entities: locations, products, contracts, coworker_invoices, coworkers, resources, extra_services

### AVA Refresh

- `functions/ava_refresh.py`
- runs `EXEC ava.sp_refresh_product_availability`
- logs before and after row count

### Xero OAuth and Sync

- `shared/xero/flow.py`
  - `start_auth()`
  - `handle_callback()`
- `shared/xero/client.py`
  - auto-refreshes tokens near expiry
  - marks connection disconnected on `invalid_grant`
- `shared/xero/invoice_sync.py`
  - incremental by tenant using `If-Modified-Since`
  - updates `meta.xero_tenants` watermarks
  - refreshes `silver.xero_tenants` after invoice sync
  - `cache_missing_pdfs()` runs after sync — fetches PDFs for any invoice with no `pdf_blob_path`
  - reuses the same retry/throttle flow as `scripts/python_scripts/backfill_xero_pdfs.py`
  - does not use `RunTracker`
- `shared/xero/tenant_directory.py`
  - matches legal Xero tenant names to Nexudus locations
  - preserves any manually maintained `community_manager_name`

### Xero PDF Storage

- PDFs are stored in Azure Blob Storage, not in SQL
- container: `xero-invoice-pdfs` on `staccinfinitspaceprod001`
- blob path format: `{xero_tenant_id}/{yyyy}/{mm}/{invoice_source_id}.pdf`
- `bronze.xero_invoice_pdfs.blob_path` and `silver.xero_invoices.pdf_blob_path` hold the reference
- `BlobWriter.write_pdf()` uploads; `BlobWriter.read_pdf()` downloads by path
- `pdf_blob_path IS NULL` is the natural watermark — only invoices still missing a cached PDF are fetched each night

### Xero ↔ Nexudus Invoice Linking

- `shared/integrations/xero_nexudus_overdue.py` — pure linking logic, no I/O
- `silver.xero_overdue_invoice_contacts` — SQL view, ready to query
- match priority: `invoice_number` > `payment_reference` > `xero_reference` > `xero_reference_payment_reference`
- same-location matches ranked above cross-location matches
- `recipient_email` coalesces: `billing_email` → `email` → `coworker_billing_email`
- rows with `match_reason = 'unmatched'` have no Nexudus record; `recipient_email` will be NULL
- current coverage: all 12 Xero tenants connected (Starter tier limit)

---

## Logging and Operational Expectations

### RunTracker-backed functions

`RunTracker` writes to `meta.sync_runs` for:

- Nexudus bronze entity runs (including coworker_invoices, coworkers)
- Nexudus silver worker entity runs
- AVA refresh

Expected SQL status fields:

- `status`
- `started_at`
- `finished_at`
- `rows_read`
- `rows_written`
- `rows_skipped`
- `error_message`

### Expected Nexudus logs

- `Nexudus -> Bronze sync started`
- `Locations: X fetched, Y written to bronze`
- `Products: X fetched, Y written to bronze`
- `Contracts: X fetched, Y written to bronze`
- `Coworker invoices: X fetched, Y written to bronze. Distinct coworker ids: Z`
- `Coworkers: X attempted, Y written, Z skipped`
- `Resources: X attempted, Y written, Z skipped`
- `Extra services: X fetched, Y written to bronze`
- `Nexudus -> Bronze sync complete`

### Expected silver logs

- `Bronze -> Silver orchestrator started`
- `Bronze -> Silver: 7 tasks enqueued`
- `Silver worker received: entity=... dequeue_count=...`
- `Silver worker complete: entity=... result=...`

### Expected AVA logs

- `AVA refresh started`
- `AVA refresh complete: before rows -> after rows`

### Expected Xero logs

- `Xero invoice sync started`
- `Fetching Xero invoices page`
- `Writing Xero invoices page`
- `Xero invoice sync complete`
- `Xero PDF cache complete: {pdfs_cached: N, pdfs_failed: N, pdfs_total: N}`
- possible warning: `Some tenants failed during Xero sync`
- final Xero sync stats include nested `tenant_directory` refresh results

---

## Environment Variables

```bash
# Nexudus
NEXUDUS_USERNAME=...
NEXUDUS_PASSWORD=...
NEXUDUS_BEARER_TOKEN=...

# Azure SQL
AZURE_SQL_CONNECTION_STRING=...
AZURE_SQL_SERVER=...
AZURE_SQL_DATABASE=...
AZURE_SQL_USERNAME=...
AZURE_SQL_PASSWORD=...
AZURE_SQL_DRIVER="ODBC Driver 18 for SQL Server"
AZURE_SQL_CONNECTION_TIMEOUT=60
AZURE_SQL_TRUST_SERVER_CERTIFICATE=false

# Blob storage
AZURE_STORAGE_ACCOUNT_NAME=staccinfinitspaceprod001
AZURE_STORAGE_CONTAINER_RAW_NEXUDUS=nexudus-raw-snapshots
AZURE_STORAGE_CONTAINER_XERO_PDFS=xero-invoice-pdfs

# Queue trigger storage
AzureWebJobsStorage=...

# BambooHR
BAMBOOHR_SUBDOMAIN=infinitspace
BAMBOOHR_API_KEY=...
BAMBOOHR_SYNC_SCHEDULE="0 0 5 * * *"  # optional override

# Google Maps
GOOGLE_MAPS_API_KEY=...

# Xero
XERO_CLIENT_ID=...
XERO_CLIENT_SECRET=...
XERO_REDIRECT_URI=https://...
XERO_POST_AUTH_REDIRECT_URI=...
XERO_SCOPES="offline_access accounting.invoices accounting.payments ..."
INTEGRATIONS_ENCRYPTION_KEY=...

# Reply.io
REPLY_IO_API_KEY=...

# Function registration
ENABLE_ETL_FUNCTIONS=1
ENABLE_ADMIN_FUNCTIONS=0

# Schedule overrides
NEXUDUS_SYNC_SCHEDULE="0 0 2 * * *"
SILVER_SYNC_SCHEDULE="0 30 2 * * *"
AVA_REFRESH_SCHEDULE="0 0 3 * * *"
XERO_INVOICE_SYNC_SCHEDULE="0 0 4 * * *"
XERO_INVOICE_SYNC_FORCE_FULL=0
REPLYIO_SYNC_SCHEDULE="0 30 5 * * *"
```

---

## Local Validation

Recommended order:

```powershell
.\venv\Scripts\python.exe scripts\python_scripts\test_local.py --step auth
.\venv\Scripts\python.exe scripts\python_scripts\test_local.py --step sql
.\venv\Scripts\python.exe scripts\python_scripts\test_local.py --step all --dry-run --limit 20
.\venv\Scripts\python.exe scripts\python_scripts\test_local.py --step all --limit 50
.\venv\Scripts\python.exe scripts\python_scripts\test_locations_silver.py --write
.\venv\Scripts\python.exe scripts\python_scripts\test_products_silver.py --write
.\venv\Scripts\python.exe scripts\python_scripts\test_contracts_silver.py --write
.\venv\Scripts\python.exe scripts\python_scripts\test_extra_services_silver.py --write
```

Nexudus billing validation:

```powershell
# Dry-run first
.\venv\Scripts\python.exe scripts\python_scripts\sync_nexudus_billing.py --dry-run --limit 5
# Full backfill (first time only)
.\venv\Scripts\python.exe scripts\python_scripts\sync_nexudus_billing.py
# Incremental (subsequent runs)
.\venv\Scripts\python.exe scripts\python_scripts\sync_nexudus_billing.py --since-last-run
# Check Xero <-> Nexudus match rate
.\venv\Scripts\python.exe scripts\python_scripts\xero_nexudus_link_audit.py --show-unmatched
```

Xero PDF validation:

```powershell
# Test single PDF fetch (saves to disk)
.\venv\Scripts\python.exe scripts\python_scripts\test_xero_pdf.py
# Test full round-trip: fetch -> blob upload -> SQL -> read back
.\venv\Scripts\python.exe scripts\python_scripts\test_xero_pdf_cache.py
# Backfill PDFs for all existing invoices missing `pdf_blob_path`
.\venv\Scripts\python.exe scripts\python_scripts\backfill_xero_pdfs.py --dry-run
.\venv\Scripts\python.exe scripts\python_scripts\backfill_xero_pdfs.py
```

Xero validation:

```powershell
.\venv\Scripts\python.exe scripts\python_scripts\xero_start_oauth.py --owner-type workspace --owner-id default
.\venv\Scripts\python.exe scripts\python_scripts\xero_complete_oauth.py --redirect-url "<full redirect url>"
.\venv\Scripts\python.exe scripts\python_scripts\xero_get_connections.py --owner-type workspace --owner-id default
.\venv\Scripts\python.exe scripts\python_scripts\xero_sync_invoices.py --owner-type workspace --owner-id default
.\venv\Scripts\python.exe scripts\python_scripts\xero_list_invoices.py --owner-type workspace --owner-id default --top 20
.\venv\Scripts\python.exe -m unittest tests.test_xero_integration tests.test_xero_tenant_directory tests.test_xero_nexudus_invoice_linking
```

Explicit refresh verification:

1. set `meta.xero_connections.expires_at` into the past
2. run `xero_get_connections.py`
3. verify `expires_at` moved forward and `is_connected = 1`

---

## SQL Validation Queries

```sql
SELECT TOP 20
    source_name, entity, layer, status,
    started_at, finished_at,
    rows_read, rows_written, rows_skipped, error_message
FROM meta.sync_runs
ORDER BY started_at DESC;

SELECT TOP 20 * FROM meta.sync_errors ORDER BY created_at DESC;

SELECT id, owner_type, owner_id, is_connected, last_error, expires_at, updated_at
FROM meta.xero_connections
ORDER BY updated_at DESC;

SELECT
    tenant_name,
    last_invoice_sync_started_at,
    last_invoice_sync_completed_at,
    last_invoice_sync_error,
    last_invoice_modified_utc
FROM meta.xero_tenants
ORDER BY tenant_name;

SELECT
    tenant_name,
    location_name,
    location_city,
    location_country_name,
    community_manager_name,
    location_match_rule
FROM xero.silver_tenants
ORDER BY tenant_name;

-- Xero <-> Nexudus link quality
SELECT match_reason, COUNT(*) AS cnt
FROM silver.xero_overdue_invoice_contacts
GROUP BY match_reason ORDER BY cnt DESC;

-- Overdue invoices ready for email automation
SELECT invoice_number, contact_name, due_date, amount_due,
       recipient_email, coworker_full_name, match_reason
FROM silver.xero_overdue_invoice_contacts
WHERE recipient_email IS NOT NULL
  AND match_reason != 'unmatched'
ORDER BY due_date ASC;

-- PDF cache status
SELECT
    COUNT(*) AS total_overdue,
    SUM(CASE WHEN pdf_blob_path IS NOT NULL THEN 1 ELSE 0 END) AS pdfs_cached,
    SUM(CASE WHEN pdf_blob_path IS NULL THEN 1 ELSE 0 END) AS pdfs_missing
FROM silver.xero_invoices
WHERE invoice_status = 'AUTHORISED'
  AND amount_due > 0
  AND due_date < CAST(GETUTCDATE() AS DATE);
```

---

## Deployment Notes

Before deploying, ensure the new env var is set:

```powershell
az functionapp config appsettings set `
  --resource-group infinitspace-prod-northeurope-data-rg `
  --name func-infinitspace-etl `
  --settings AZURE_STORAGE_CONTAINER_XERO_PDFS=xero-invoice-pdfs
```

ETL app:

```powershell
func azure functionapp publish func-infinitspace-etl --python

az functionapp config appsettings set `
  --resource-group infinitspace-prod-northeurope-data-rg `
  --name func-infinitspace-etl `
  --settings `
    ENABLE_ETL_FUNCTIONS=1 `
    ENABLE_ADMIN_FUNCTIONS=0
```

Optional admin app:

```powershell
func azure functionapp publish func-infinitspace-etl --python

az functionapp config appsettings set `
  --resource-group infinitspace-prod-northeurope-data-rg `
  --name func-infinitspace-etl `
  --settings `
    ENABLE_ETL_FUNCTIONS=0 `
    ENABLE_ADMIN_FUNCTIONS=1
```

---

## Current Status
 
| Feature | Status | Notes |
|---------|--------|-------|
| Nexudus bronze sync | done | 7 entities (added coworker_invoices + coworkers) |
| Nexudus silver fanout | done | queue-based, 7 entities |
| AVA refresh | done | stored procedure rebuild |
| Xero OAuth + tenant storage | done | DB-backed |
| Xero auto-refresh | done | disconnects on `invalid_grant` |
| Xero invoice sync | done | incremental by tenant |
| Xero tenant directory | done | refreshed after Xero sync and exposed as `xero.silver_tenants` |
| Xero invoice PDF caching | done | blob storage (`xero-invoice-pdfs`); path in `silver.xero_invoices.pdf_blob_path`; auto-cached nightly for invoices missing `pdf_blob_path` |
| Nexudus coworker invoices + coworkers | done | incremental via UpdatedSince watermark |
| Xero ↔ Nexudus invoice linking | done | `silver.xero_overdue_invoice_contacts` view; 5/12 tenants connected |
| Optional admin HTTP routes | done | separate deployment mode |
| Google Maps scheduled pipeline | not wired | utilities exist, not registered in default app |
| Core layer population | planned | not implemented |
| Real Estate CoStar extractor HTTP function | done | `ENABLE_REAL_ESTATE_FUNCTIONS=1` to activate |
| BambooHR employee sync | done | bronze + silver; `work_email` is join key to Nexudus coworkers |
| Reply.io stats sync | done | bronze only; sequence steps + daily step performance; 4 AB test sequences |

---

## Self-Update Rules

After any material project change:

1. update repository structure if files moved or were added
2. update function registry if triggers changed
3. update env vars if configuration changed
4. update runtime topology if schedules or dependencies changed
5. update validation steps if the recommended test flow changed
6. update current status if a feature moved from planned to done or vice versa
7. always update the date at the bottom

---

Last updated: 2026-04-14 (gold finance dashboard: switched due_date/amount_due to Nexudus source, excluded unmatched invoices, added due_amount>0 and due_date>=2026-03-01 filters; all 12 Xero tenants now connected)
Current branch: `main`
Maintainer: InfinitSpace Data Engineering Team

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Infinitspace1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
