## onto-demo

> * Warehouse: runbook specifies `310314146ebd3230`, but it does not exist in

# RT_onto project rules

* Warehouse: runbook specifies `310314146ebd3230`, but it does not exist in
  `jai_classic_ws`; operator directed use of `bf7ffcda00a8c351` (jai-sql-warehouse).
  Warehouse is compute only — it does not change where artifacts live.
* Use `jai_ontos`.`demo_schema` for all created workshop artifacts.
* Every created object name must start with `jai_`.
* Keep the ontology and business vocabulary stable even if physical table names differ.
* Prefer mapping local physical schema to business concepts over recreating a source-system namespace.
* Only create synthetic seed data if the local schema lacks usable rows for the app.
* Keep business KPI logic centralized in `ontology/semantic_measures.yaml` and SQL serving views.
* Build in phases. Do not skip directly to enterprise-wide automation before the single use case works.
* The app must support one use case first and multi-product selection second.
* Store repeatable task instructions as local skills under `skills/`.

## Environment

| Setting | Value |
| --- | --- |
| Workspace | `jai_classic_ws` (CLI profile `jai-classic`) |
| Source-of-truth inventory | `inputs/describe_table_extended.csv` |
| Source reference catalog | `fc_entdata_gold` (`cdm_dim`, `cdm_bi`, `cdm_pos`) |
| Local physical catalog/schema | `jai_ontos.rt_str_lbr` |
| Artifact catalog/schema | `jai_ontos.demo_schema` |
| Warehouse | `310314146ebd3230` |
| App | `RT_onto_demo` (Streamlit) |

## First use case

Store Traffic & Labor Efficiency → product `jai_store_traffic_labor_efficiency`.
Concepts: Store, CalendarDay, StoreTrafficDay, StoreLaborDay, StoreDayEfficiency.

## Layout

`docs/` reports · `ontology/` semantic model + mappings · `ontobricks/` ontology
project (design, mapping, materialization, queries) · `ontos/` governed artifacts
(products, contracts, semantic models) · `sql/` serving views (`jai_`-prefixed) ·
`app/` the Streamlit app · `skills/` repeatable task instructions · `tests/` checks.

## How to run

* Live: `databricks auth login -p jai-classic`, then run `sql/01..04` on the
  warehouse, then `streamlit run app/app.py` with `RT_ONTO_DATA_MODE=warehouse`.
* Offline demo: `python app/seed/generate_seed.py` then `streamlit run app/app.py`
  (defaults to `RT_ONTO_DATA_MODE=seed`).

---
> Source: [jjaiwant328/onto-demo](https://github.com/jjaiwant328/onto-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
