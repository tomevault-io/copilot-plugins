## rds-skill

> Query any AWS Aurora PostgreSQL cluster via IAM authentication and psycopg2. Use whenever the user mentions RDS, Aurora, PostgreSQL, Aurora PgSQL, database queries, schema exploration, table metadata, column listing, data profiling, or wants to explore Aurora database objects. Also use for local analytics on previously saved query results. Works with any Aurora PostgreSQL cluster with IAM database authentication enabled. Always use this skill for any Aurora PostgreSQL task, even simple SELECT queries.


# Aurora PostgreSQL Skill

Read-only Aurora PostgreSQL exploration and business analysis via IAM authentication. No passwords or secrets required — your AWS IAM identity is your credential. Cross-platform (Mac + Windows). Works with any AI coding agent.

All scripts are in `${CLAUDE_SKILL_DIR}/scripts/` and require Python 3, AWS CLI, and psycopg2.

## Python Command

Read `~/.rds-skill/config.json` and use the `"python"` key as the Python command.
If config doesn't exist yet, try `python3 --version` first, falling back to `python --version`.
Throughout this document, `PYTHON` means the detected Python command.

## First-Time Setup

**You cannot run the setup wizard directly** — it requires interactive terminal input.

Check if `~/.rds-skill/config.json` exists:
- **If it exists:** Read it to discover the configured connections (the `connections` map and the `default` key).
- **If it doesn't exist:** Tell the user to run the setup wizard in their terminal:

> Run this in your terminal to configure the Aurora connection:
> ```
> python3 scripts/setup.py
> ```
> (On Windows, use `python` instead of `python3`)

Wait for the user to confirm setup is complete before running any queries.

## Connection Selection

The skill supports **multiple named connections** (e.g. prod, staging, local). Config shape:

```json
{
  "default": "prod",
  "connections": {
    "prod": { "host": "...", "database": "main", "db_user": "...", "region": "eu-west-1", "profile": "...", "write_mode": "reject" },
    "staging": { "...": "..." }
  },
  "python": "/usr/bin/python3"
}
```

- Every script accepts `--connection NAME` to pick a specific connection.
- Without `--connection`, scripts use the `default` connection.
- To list connections: `PYTHON ${CLAUDE_SKILL_DIR}/scripts/setup.py --list`
- To switch defaults: `PYTHON ${CLAUDE_SKILL_DIR}/scripts/setup.py --set-default NAME`
- To remove one: `PYTHON ${CLAUDE_SKILL_DIR}/scripts/setup.py --remove NAME`
- To add another: re-run `PYTHON ${CLAUDE_SKILL_DIR}/scripts/setup.py` (interactive — instruct the user to run it themselves).

When the user mentions "prod" / "staging" / a specific cluster name, map that to the matching connection and pass `--connection NAME` on every script invocation. If they don't specify, use the default and mention which one you're using.

## Quick Reference

| Task | Script | When to use | Key Args |
|------|--------|-------------|----------|
| **Run SQL** | `query.py` | Any free-form read-only query | `"SELECT ..."` or `--sql-file=PATH` |
| **List schemas** | `schemas.py` | Starting point — see what schemas exist with table counts | |
| **List tables** | `tables.py` | Browse tables, check row counts and sizes before querying | `--schema=NAME` |
| **List columns** | `columns.py` | Understand column types, nullability, indexes | `--schema=NAME --table=NAME` |
| **Search objects** | `search.py` | Find tables or columns when you don't know the exact name | `--pattern=TEXT` |
| **Sample data** | `sample.py` | Quick peek at actual values — always do this before writing queries | `--schema=NAME --table=NAME` |
| **Data profile** | `profile.py` | Per-column stats (nulls, cardinality, min/max/avg) | `--schema=NAME --table=NAME` |
| **Local analytics** | `analyze.py` | Analyze saved results locally without hitting Aurora | `FILE --describe` |

### Common options (all RDS scripts)

| Option | Description |
|--------|-------------|
| `--format=txt\|csv\|json` | Terminal display format (default: txt) |
| `--save-format=txt\|csv\|json` | File save format (default: csv) |
| `--save=PATH` | Save to a specific file path |
| `--no-save` | Don't auto-save to ~/rds-exports/ |
| `--save-sql` | Save the SQL query as a .sql file alongside results |
| `--sql-file=PATH` | Read SQL from a file (query.py only) |
| `--connection=NAME` | Pick a named connection from `~/.rds-skill/config.json` (defaults to the saved `default`) |
| `--profile=NAME` | Override AWS profile |
| `--host=HOST` | Override cluster endpoint |
| `--database=NAME` | Override database |
| `--db-user=NAME` | Override database user |
| `--timeout=N` | Max wait seconds (default: 120) |
| `--max-rows=N` | Max rows to fetch (default: 1000) |

## Output and File Saving

All query results are **automatically saved** to `~/rds-exports/query-{timestamp}.csv`.
The terminal shows an aligned txt preview (first 200 rows). The saved file defaults to CSV for spreadsheet compatibility.

This means you always have:
- **Inline preview** (200 rows in txt format) — enough to understand the data shape and answer quick questions
- **Full CSV on disk** — for deeper analysis with `analyze.py` or for the user to open in a spreadsheet

`--format` controls terminal display (default: txt). `--save-format` controls the saved file format (default: csv). Use `--save-sql` to also save the SQL query as a matching `.sql` file. Use `--no-save` to skip auto-save. Use `--save=PATH` to save to a specific location.

---

## Defensive Guardrails

These rules protect the cluster from expensive queries. Follow them — but use judgement. If the user explicitly asks for something that bends a rule, explain the trade-off and proceed if they confirm.

- **Never `SELECT *` from tables with >10K rows** — use aggregations, filters, or `sample.py` instead. For smaller tables, `SELECT *` is fine.
- **Always add `LIMIT`** when exploring unfamiliar tables (default LIMIT 100).
- **Check row counts first** — run `tables.py --schema=X` before writing queries so you know what you're dealing with.
- **Prefer aggregations for large tables** — `COUNT`, `SUM`, `AVG` with `GROUP BY` over pulling raw rows.
- **Filter on indexed columns for large tables** — check indexes via `columns.py` (the `indexes` column). Filtering on indexed columns avoids full table scans.
- **Joins are fine** — as long as the join condition is correct and you aggregate/filter the result appropriately.
- **Avoid accidental cross joins** — always include `ON`/`USING`.
- **Prefer `LIMIT` + `ORDER BY`** over unbounded selects when exploring.

**Size awareness:**

| Table size | Approach |
|------------|----------|
| <10K rows | Explore freely, `SELECT *` is fine |
| 10K–1M rows | Add `WHERE` or `LIMIT`, aggregations preferred for full-table queries |
| >1M rows | Always aggregate or filter, never `SELECT *`, use indexed column filters |

---

## SQL Standards

Every SQL query you write must follow these rules.

### Always comment your SQL

Explain the business intent, not just the mechanics.

### Always show the SQL to the user

- **Short queries (<10 lines):** show the full SQL inline with comments
- **Long queries (>10 lines):** save to `~/rds-exports/query-{timestamp}.sql`, show the key parts inline, and reference the saved file

### Use `--sql-file` for complex queries

For long SQL, write it to a file first and execute with `--sql-file`:
```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/query.py --sql-file=~/rds-exports/my_query.sql
```

**Tip — inline multiline SQL without reading/writing in the LLM context:**

*Mac/Linux — heredoc:*
```bash
cat > "/path/to/query.sql" << 'EOSQL'
-- Your SQL here
SELECT ...
EOSQL
PYTHON ${CLAUDE_SKILL_DIR}/scripts/query.py --sql-file="/path/to/query.sql" --save="/path/to/results.csv" --save-sql
```

*Cross-platform — Python:*
```bash
PYTHON -c "
sql = '''
-- Your SQL here
SELECT ...
'''
open('/path/to/query.sql', 'w').write(sql.strip())
"
PYTHON ${CLAUDE_SKILL_DIR}/scripts/query.py --sql-file="/path/to/query.sql" --save="/path/to/results.csv" --save-sql
```

**Tip — reuse SQL files by replacing values:** When you already have a SQL file and just need to change specific values (e.g. rolling dates forward a week), use a one-liner to swap them without rewriting the whole file in the LLM context:

*Mac/Linux — sed:*
```bash
sed "s/2026-03-15/2026-03-22/g; s/2026-03-21/2026-03-28/g" original_query.sql > query.sql
```

*Cross-platform — Python:*
```bash
PYTHON -c "open('query.sql','w').write(open('original_query.sql').read().replace('2026-03-15','2026-03-22').replace('2026-03-21','2026-03-28'))"
```

Then execute:
```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/query.py --sql-file=query.sql --save="/path/to/results.csv" --save-sql
```
For templates with many replacements, named placeholders like `{{WC_START}}` can be used instead of literal dates.

### Formatting conventions

```sql
------------------------------------------------------------------------------------------------------------------------
-- Monthly revenue by region
-- Purpose: Aggregate order line items by region for the last 12 months
-- Filters: Excludes cancelled orders and zero-quantity items
------------------------------------------------------------------------------------------------------------------------
WITH monthly_revenue AS (
    SELECT  r.region_name,
            DATE_TRUNC('month', o.order_date)::DATE                                  AS month_nk,
            SUM(o.quantity)                                                           AS total_quantity,
            SUM(o.line_total)                                                         AS total_revenue,
            COUNT(DISTINCT o.customer_id)                                             AS unique_customers
    FROM    sales.fact_order_lines o
    JOIN    sales.dim_regions r                     USING (region_id)
    WHERE   o.order_date >= NOW() - INTERVAL '12 months'
            AND o.quantity > 0
            AND o.order_status <> 'cancelled'
    GROUP   BY 1, 2
)
SELECT  region_name,
        month_nk,
        total_quantity,
        total_revenue,
        unique_customers,
        ROUND(total_revenue / NULLIF(total_quantity, 0), 2)                          AS revenue_per_unit
FROM    monthly_revenue
ORDER   BY region_name, month_nk
```

Key formatting rules:
- Section headers with `--` dashes above the query
- `SELECT`, `FROM`, `JOIN`, `WHERE`, `GROUP BY`, `ORDER BY` left-aligned
- **One column per line** — each column expression gets its own line with its `AS` alias
- Column aliases aligned at a consistent position
- Inline comments explaining non-obvious business logic
- CTE names that describe the business concept

### PostgreSQL-specific SQL notes

- Use `NOW() - INTERVAL '12 months'` for date arithmetic (not `DATEADD`)
- Use `EXTRACT(epoch FROM timestamp)` for epoch seconds
- Use `DATE_TRUNC('month', col)` for period truncation
- Use `NULLIF(expr, 0)` to avoid divide-by-zero
- `GROUP BY 1, 2, 3` positional references work in PostgreSQL
- Use `ILIKE` for case-insensitive pattern matching
- Use `::TYPE` for casting (e.g. `col::DATE`, `col::TEXT`, `col::NUMERIC`)

### Generous commenting

Every query must include comments that explain the **why**, not just the what. Treat SQL as documentation for the next person (or the next agent) who reads it.

- **Header block** at the top of every query: purpose, sources, key assumptions, parameters
- **Section separators** (dashed `--` lines) between logical blocks (e.g. between actuals, budget, pivot CTEs)
- **Inline comments** on non-obvious expressions (e.g. customer estimation formulas, business rules, why a filter exists)
- **CTE-level comments** explaining what each CTE produces and why it exists as a separate step

```sql
------------------------------------------------------------------------------------------------------------------------
-- Weekly Sales Scorecard
-- Purpose: Core trading metrics with TY/LP/LY comparisons and BU/LV variances
-- Source:  fact_basket_items (actuals), fact_commercial_budget_all (BU)
-- Params:  Change the date in the periods CTE to set the reporting period
-- Notes:
--   - Customer estimation done per channel first, then summed to total
--   - BU/LV only have sales and margin — derivative ratios are actuals-only
------------------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------------------------------------------------------
-- ACTUALS: per channel for correct customer estimation
------------------------------------------------------------------------------------------------------------------------
```

---

## Schema Exploration Workflow

Use this graduated approach when exploring an unfamiliar schema. **Don't follow this rigidly** — if the user's intent is clear, skip straight to the relevant step.

1. **Understand the landscape** — `schemas.py` → see what schemas exist
2. **Browse tables** — `tables.py --schema=X` → check row counts and sizes
3. **Understand structure** — `columns.py --schema=X --table=Y` → column types, indexes
4. **Sample first** — `sample.py --limit=5` → see actual data values
5. **Profile if needed** — `profile.py` → nulls, cardinality, min/max per column
6. **Write targeted queries** — now you know enough to write safe, efficient SQL
7. **Analyze locally** — `analyze.py` on saved results for follow-up

**Shortcut:** If the user says "how many orders last month?", don't run 5 discovery scripts. Check the table size, write the query, run it.

---

## Business Analysis Workflows

### Understanding business performance

1. **Start with the headline metric** — total revenue, order count, customer count for the period
2. **Break down by dimensions** — time (daily/weekly/monthly), region, channel, category
3. **Compare periods** — this period vs last period, year-over-year
4. **Identify outliers** — which segments are significantly above or below expectations?

### Root cause analysis

When something looks wrong:
1. **Confirm the anomaly** — is it real? Check the data source
2. **Decompose the metric** — volume vs value issue?
3. **Slice by dimensions** — which region/channel/category drove the change?
4. **Correlate with events** — promotions, price changes, stock-outs, seasonality

### Common analyst patterns

| Pattern | Approach | SQL shape |
|---------|----------|-----------|
| **Trend analysis** | Track metrics over time | `GROUP BY DATE_TRUNC('month', col)` + `SUM`/`COUNT` |
| **Cohort analysis** | Group by first purchase | `MIN(order_date)` per customer, then join back |
| **Top/bottom N** | Best and worst performers | `ORDER BY metric DESC LIMIT N` |
| **YoY comparison** | Year-over-year growth | `LAG()` window or self-join with date shifted 1 year |
| **Funnel analysis** | Conversion at each step | `COUNT(DISTINCT user_id)` per step |
| **Contribution** | Which items drive 80% of revenue | `SUM() OVER (ORDER BY ...)` cumulative |
| **Segmentation** | Group entities by behavior | `CASE WHEN` buckets, then profile each segment |

---

## Script Details

### query.py — Run SQL

```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/query.py "SELECT count(1) FROM sales.orders"
PYTHON ${CLAUDE_SKILL_DIR}/scripts/query.py --sql-file=~/rds-exports/my_query.sql
```

### schemas.py — List schemas

```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/schemas.py
```
```
schema_name   owner   table_count
-----------   -----   -----------
public        admin   12
sales         etl     45
```

### tables.py — List tables

```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/tables.py --schema=sales
```
```
table_name       table_type  row_count  total_size  table_size  index_size
--------------   ----------  ---------  ----------  ----------  ----------
dim_customers    BASE TABLE  250000     120 MB      85 MB       35 MB
fact_orders      BASE TABLE  8500000    4500 MB     3800 MB     700 MB
```

### columns.py — List columns

```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/columns.py --schema=sales --table=orders
```
```
pos  column_name   data_type          max_len  is_nullable  column_default  indexes
---  ------------  -----------------  -------  -----------  --------------  -------
1    order_id      INTEGER                     NO           nextval(...)    orders_pkey
2    order_date    TIMESTAMP WITHOUT           NO                           idx_orders_date
3    customer_id   INTEGER                     NO                           idx_orders_cust
```

### search.py — Search objects

```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/search.py --pattern=order
PYTHON ${CLAUDE_SKILL_DIR}/scripts/search.py --pattern=revenue --type=column
```

### sample.py — Sample data

```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/sample.py --schema=sales --table=orders --limit=5
```

### profile.py — Data profiling

```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/profile.py --schema=sales --table=orders
```
```
column_name   data_type  total_rows  null_count  null_pct  distinct_count  min_val     max_val     avg_val
-----------   ---------  ----------  ----------  --------  --------------  ----------  ----------  -------
order_id      INTEGER    8500000     0           0.0       8500000         1           8500000     4250000
order_date    TIMESTAMP  8500000     0           0.0       1825            2022-01-01  2026-12-31
```

### analyze.py — Local analytics (no Aurora)

Analyze previously saved CSV/JSON files locally. No network access needed — great for follow-up analysis without hitting Aurora again.

```bash
PYTHON ${CLAUDE_SKILL_DIR}/scripts/analyze.py ~/rds-exports/query-*.csv --describe
PYTHON ${CLAUDE_SKILL_DIR}/scripts/analyze.py data.csv --sum=revenue
PYTHON ${CLAUDE_SKILL_DIR}/scripts/analyze.py data.csv --group-by=region --sum=sales
PYTHON ${CLAUDE_SKILL_DIR}/scripts/analyze.py data.csv --filter='year=2024' --sort=amount --desc --top=10
PYTHON ${CLAUDE_SKILL_DIR}/scripts/analyze.py data.csv --hist=price
```

Available operations: `--count`, `--describe`, `--sum=COL`, `--avg=COL`, `--min=COL`, `--max=COL`, `--median=COL`, `--group-by=COL`, `--filter=EXPR`, `--sort=COL`, `--desc`, `--top=N`, `--hist=COL`

---

## Write-Mode Behaviour

Each connection has a `write_mode` field (`reject` / `accept` / `ask` / `auto`) that controls whether non-read-only SQL is allowed. Read it from the connection's config entry **before** writing or running any SQL — your behaviour changes per mode.

### Operation classification

| Class | Examples |
|---|---|
| **Read** | `SELECT`, `WITH`, `SHOW`, `EXPLAIN`, `SET` |
| **Low-risk write** | `INSERT INTO ... VALUES`, `INSERT INTO ... SELECT`, `UPDATE ... WHERE`, `DELETE ... WHERE`, `CREATE TABLE/VIEW/INDEX`, `COMMENT ON`, `ANALYZE`, `VACUUM` |
| **High-risk write** | `DROP`, `TRUNCATE`, `UPDATE` without `WHERE`, `DELETE` without `WHERE`, `ALTER TABLE ... DROP`, `GRANT`, `REVOKE`, `CREATE OR REPLACE` on existing objects |

### Behaviour matrix

| `write_mode` | Read | Low-risk write | High-risk write |
|---|---|---|---|
| `reject` | run | **script blocks** — refuse to even submit | **script blocks** |
| `auto` | run | run | **stop and ask the user before submitting** |
| `ask` | run | **stop and ask the user before submitting** | **stop and ask the user before submitting** |
| `accept` | run | run | run (no prompt) |

### How to "stop and ask"

When the matrix says to ask the user **before** running the query (this happens at the LLM level, not the script — by the time the script runs, the answer is already yes):

1. Compose the SQL.
2. Use the agent's structured-question tool if available — `AskUserQuestion` in Claude Code, equivalent prompt tools in Codex / Cursor / etc. — rather than free-text Q&A.
3. The question must show:
   - the SQL (formatted),
   - the connection name + database,
   - the target objects,
   - a blast-radius estimate (rows affected, whether reversible).
4. Only proceed on an explicit yes. On no, abort.

Example confirm question for `DELETE FROM events WHERE created_at < '2024-01-01'` on the `prod` connection (write_mode = `auto`, low-risk because it has a WHERE):

> *Connection `prod` is in `auto` mode and this is a low-risk write — running directly. (No confirmation needed.)*

Example for `DELETE FROM events` (no WHERE) on the same connection:

> *Connection `prod` (write_mode `auto`) — this is a high-risk write. Confirm before I run:*
> ```sql
> DELETE FROM events
> ```
> *Estimated rows affected: ~12.3M (entire table). Irreversible. Run? [yes / no]*

### Multi-statement queries

Multi-statement queries (`;` followed by another statement) are blocked **in all write modes** — that's an injection defence, not a read-only thing.

### Defensive defaults

If you can't determine the write_mode (config missing, malformed), assume `reject` and only run reads.

---

## How IAM Authentication Works

This skill uses **AWS IAM database authentication** — no passwords or secrets are stored anywhere.

1. Your AWS CLI profile (`de_rds` or similar) provides your identity
2. The skill calls `aws rds generate-db-auth-token` to get a temporary 15-minute token
3. The token is used as the database password over an SSL-encrypted connection (`sslmode=require`)
4. The token expires automatically — no credential rotation needed

**Prerequisites (one-time setup, done by infra):**
- Aurora cluster: `iam_database_authentication_enabled = true`
- DB user: `GRANT rds_iam TO rds_skill_user`
- IAM policy: `rds-db:connect` on the cluster + user ARN
- VPN: connected to corporate VPN to reach the DB endpoint

---

## Advanced SQL Templates

For ad-hoc exploration via `query.py`. PostgreSQL system catalogs — no extensions or admin views required (except `pg_stat_statements` where noted).

**Running queries:**
```sql
SELECT pid,
       usename                                                       AS user_name,
       datname                                                       AS db_name,
       state,
       query_start,
       NOW() - query_start                                           AS duration,
       LEFT(query, 200)                                              AS query_preview
FROM   pg_stat_activity
WHERE  state <> 'idle'
       AND pid <> pg_backend_pid()
ORDER  BY query_start
```

**Long-running / blocking queries:**
```sql
SELECT pid,
       usename,
       state,
       wait_event_type,
       wait_event,
       NOW() - query_start                                           AS duration,
       LEFT(query, 200)                                              AS query_preview
FROM   pg_stat_activity
WHERE  state = 'active'
       AND NOW() - query_start > INTERVAL '30 seconds'
ORDER  BY query_start
```

**Query history (requires `pg_stat_statements` extension):**
```sql
SELECT LEFT(query, 200)                                              AS sql,
       calls,
       ROUND(total_exec_time::NUMERIC, 1)                            AS total_ms,
       ROUND(mean_exec_time::NUMERIC, 1)                             AS mean_ms,
       ROUND((100 * total_exec_time / SUM(total_exec_time) OVER ())::NUMERIC, 1)  AS pct_of_total
FROM   pg_stat_statements
ORDER  BY total_exec_time DESC
LIMIT  20
```
If `pg_stat_statements` isn't installed, this query will error with `relation "pg_stat_statements" does not exist` — that's a one-time install at the cluster level (`CREATE EXTENSION pg_stat_statements`).

**Table dependencies (FK relationships):**
```sql
SELECT n1.nspname || '.' || c1.relname                               AS source_table,
       n2.nspname || '.' || c2.relname                               AS referenced_table,
       con.conname                                                    AS constraint_name
FROM   pg_constraint con
JOIN   pg_class c1     ON con.conrelid    = c1.oid
JOIN   pg_namespace n1 ON c1.relnamespace = n1.oid
JOIN   pg_class c2     ON con.confrelid   = c2.oid
JOIN   pg_namespace n2 ON c2.relnamespace = n2.oid
WHERE  con.contype = 'f'
ORDER  BY source_table
```

**Index usage stats (which indexes are unused?):**
```sql
SELECT schemaname,
       relname                                                       AS table_name,
       indexrelname                                                  AS index_name,
       idx_scan                                                      AS times_used,
       pg_size_pretty(pg_relation_size(indexrelid))                  AS index_size
FROM   pg_stat_user_indexes
WHERE  schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER  BY idx_scan, pg_relation_size(indexrelid) DESC
```

**Table bloat estimate:**
```sql
SELECT schemaname,
       relname                                                       AS table_name,
       n_live_tup                                                    AS live_rows,
       n_dead_tup                                                    AS dead_rows,
       ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 1)  AS dead_pct,
       last_vacuum,
       last_autovacuum
FROM   pg_stat_user_tables
WHERE  n_dead_tup > 0
ORDER  BY n_dead_tup DESC
LIMIT  20
```

---
> Source: [onsen-ai/rds-skill](https://github.com/onsen-ai/rds-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
