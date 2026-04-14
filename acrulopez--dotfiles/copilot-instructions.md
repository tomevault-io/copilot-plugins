## dotfiles

> Developing dbt models

# dbt & SQL Standards

## SQL Style Guide (Strict)
- **Import CTEs:** You must use the "Import CTE" pattern for all models.
    1.  **Import:** Select from sources/refs at the very top.
    2.  **Logical:** Perform transformations in subsequent CTEs.
    3.  **Final:** Select `*` from the final CTE at the bottom.
    - *Reason:* This allows instant debugging by querying any CTE in the chain.
- **Keywords:** Use `lowercase` for all SQL keywords (`select`, `from`, `where`).
- **Trailing Commas:** Always use leading commas in select lists (easier diffs).
- **No `SELECT *`:** Explicitly list columns in all models. `SELECT *` is only allowed in intermediate CTEs if strictly necessary.

## BigQuery Optimization (Cost & Performance)
- **Partitioning:** When creating tables larger than 1GB, always suggest a `partition_by` config (usually by date/timestamp).
- **Clustering:** Always suggest `cluster_by` on high-cardinality filter columns (e.g., `user_id`, `customer_id`).
- **Data Types:** Prefer `INT64` for ID columns over `STRING` where possible for faster joins.
- **Incremental:** For large tables, prefer `incremental` materialization. Prefer `insert_overwrite` over `merge` unless the source table is partitioned by a different field than the incremental key.

## Naming Conventions
- **Files:** `snake_case` (e.g., `stg_stripe__payments.sql`). Models must be plural (e.g., `payments.sql`).
- **Columns:** If two fields in different tables refer to the same concept, use the same name for both fields.
- **Layers:**
  - `stg_[source]__[entity]`: 1:1 cleaning of source data. No joins. Cast all fields to the appropriate data types.
  - `int_[concept]`: Complex logic, joins, aggregations.
  - `fct_[concept]`: Long, thin transactional tables (Facts).
  - `dim_[concept]`: Short, wide descriptive tables (Dimensions).
- **Keys:** Primary keys must be named `[entity]_id` (e.g., `user_id`, not `id`).
- **Dates:** Use `<event>_date` for dates.
- **Timestamps:** Use `<event>_at` for timestamps.
- **Booleans:** Use `is_` or `has_` for booleans.

## Dbt Specifics
- **References:** NEVER hardcode `database.schema.table`. ALWAYS use `{{ ref('model_name') }}` or `{{ source('source', 'table') }}`.
- **Config:** Do not define materialization and tags in the model file unless it is different from the default.

## Yaml Standards
- **Materialization:** Always create a yml file per model with the same name as the model file but with the .yml extension.
- **Description:** Always add a description in the YAML for each model.
- **Use the doc() block**: If a doc block exists for a column, use it on the description (`description: '{{ doc("product_category") }}'`). If a field is repeted across multiple models, add a doc block to the model that describes the field.
- **Testing:** Every model must have:
    - `unique` and `not_null` tests on the Primary Key.
- **Use the newer dbt syntax:** Use `data_tests` instead of the deprecated `tests` field. Use `config` block inside `data_tests`, for `meta` field, etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/acrulopez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
