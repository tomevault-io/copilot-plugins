## hackhpi26

> Project structure and key references for the Cula Hack HPI hackathon project


# Cula Hack HPI — Project Context

## What this project is

A hackathon project working with the **Cula carbon removal API** — tracking biochar-based carbon sinks, their supply-chain event graphs, proof documents, and pyrolysis machine sensor data.

- the process is a **acyclic directed graph**
- the process can be looked at as a **massflow** through the steps

## Key references

- **API documentation:** `docs/API_REFERENCE.md` — full endpoint and data model reference
- **Some Machine documation:** `docs/MACHINE_ANALYSIS.md`
- **OpenAPI spec (raw JSON):** `openapi.json`
- **API base URL:** `https://api.hack-hpi.cula.earth`
- **No authentication required.** Rate limit: 60 req/min/IP.

## Process steps eventTypes:

 ExecutionType: **`material_processing or delivery-leg`**
 ```json
 1) biomass_creation **[material_processing]**
 2) biomass_processing [material_processing]
 3) biomass_delivery **[delivery-leg]**
 4) pyrolysis (this should be the machine) [material_processing]
 5) biochar_delivery_to_processing [material_processing]
 6) biochar_processing [material_processing]
 7) biochar_delivery_to_sink **[delivery-leg]**
 8) sink_creation [material_processing]
 ```


## Cula API workflow

1. `GET /api/hack-hpi/sinks` → list sink UUIDs
2. `GET /api/hack-hpi/sinks/{id}` → full sink with event graph, sites, materials, LCA data
3. `GET /api/hack-hpi/sites/{siteId}/machines` → machine UUIDs (use `carbonCaptureSiteId` from sink)
4. `GET /api/hack-hpi/machines/{machineId}/data-points` → data point config UUIDs
5. `GET /api/hack-hpi/machine-data-points/{id}` → data point metadata (name, unit, frequency)
6. `POST /api/hack-hpi/machine-data` → time-series sensor data
7. `GET /api/hack-hpi/documents/{cloudStorageId}` → download proof files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucakbb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
