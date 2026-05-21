## psql

> 1. All psql should be written in lowercase syntax

1. All psql should be written in lowercase syntax
2. When creating psql indices we should be as concise as we can, omitting defaults like the index name and the type. So we rather do `create index on table(column)` instead of `create idx_table_column on table using btree (column)`
3. We should add indices to columns that reference other tables and columns that are queried frequently. e.g. if the column is `slug` it will most likely be queried on that instead of `id` so we should create an index on it.

---
> Source: [chad-syntax/agentsmith](https://github.com/chad-syntax/agentsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
