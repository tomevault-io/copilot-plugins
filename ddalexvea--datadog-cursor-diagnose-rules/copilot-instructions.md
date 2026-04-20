## datadog-cursor-diagnose-rules

> Rules for troubleshooting DBM and APM correlation issues


# DBM and APM Correlation Troubleshooting

## Overview

APM–DBM correlation connects database spans from APM traces with query samples from Database Monitoring, enabling features like viewing database info in APM and APM data in DBM.

**Reference:** [Troubleshooting DBM and APM Correlation](https://datadoghq.atlassian.net/wiki/spaces/TS/pages/3183772859)

## Items to Collect

Before troubleshooting, obtain:

1. **APM trace link** containing a database span for the query in question
2. **DBM link** for the same exact query:
   - (ideally) A Query Sample
   - An Explain Plan
   - Confirmation whether Query Metrics exist for that query
3. **Note:** The DBM and APM query text should be exactly the same (small cosmetic differences like spaces are okay)

## Key Questions Checklist

### Question 1: Do you have a Query Sample or Explain Plan?

- **No** → No DBM sample/plan exists for this query → Go to **Bucket A**
- **Explain Plan only** → DBM has query metrics but not samples (common for SQL Server) → Go to **Bucket A/D**
- **Query Sample exists** → Go to Question 2

### Question 2: If you have a Query Sample, are there comments in `db.metadata.comments`?

Search DBM samples with:
```
@db.metadata.comments:* OR @trace.caller.service:*
```

Or open the Query sample → Event Attributes → `db.metadata.comments`

When comments are present, you should see fields like `dddbs`, `ddps`, etc.

Also check `@trace.span.resource_hash` for comparison with the APM span.

- **No comments** → Go to **Bucket B** or **Bucket D**
- **Comments present** → Go to Question 3

### Question 3: Do you have a trace with a database span, and what does the hidden metadata show?

**To view hidden span metadata (internal only):**

1. Click on the database span in the trace
2. Append `&config_trace_show_hidden_metadata=true` to the URL

⚠️ **DO NOT SHARE TRACE LINKS WITH THIS INTERNAL PARAMETER WITH CUSTOMERS**

**From the database span, record:**
- `dbm_trace_injected` → whether the tracer injected DBM comments
- `_dd.agent_version` → Agent version
- `_dd.tracer_version` → Tracer version
- `component` → DB library being traced
- `language` → application language
- `_dd.query_signature` → used for statement-based correlation

## Root Cause Buckets

---

### Bucket A: No Query Sample or Explain Plan Exists

**Scenario:** Question 1 = "No" (no sample or plan at all)

**Likely Causes:**
- Query is too infrequent or too fast → DBM doesn't sample it (sampling is biased to high-load/slow queries)
- DBM is not correctly configured or enabled for that database
- For SQL Server: `lookback_window` for query metrics may be too short

**What to Check:**

1. Confirm DBM requirements: https://docs.datadoghq.com/database_monitoring/connect_dbm_and_apm/?tab=go#before-you-begin
2. Verify Agent version and DBM configuration for that database
3. For SQL Server, consider increasing `lookback_window` (especially for infrequent queries)
4. Check Agent flare for errors in query sample / activity collection

**Escalation:** DBM / #support-database-monitoring

---

### Bucket B: DBM has Sample/Plan but No Comments and/or No DB Span

**Scenario:**
- Question 1 = "Yes" (sample/plan exists)
- Question 2 = "No" (no comments) AND/OR
- Question 3 = "No DB span" (no DB span in APM)

**Interpretation:**
The DB query is observed by DBM, but:
- The APM tracer did not create DB spans, OR
- DB spans exist but did not inject comments into the SQL

**What to Check:**

1. Confirm APM tracing & DBM–APM requirements:
   - Tracing collection: https://docs.datadoghq.com/tracing/trace_collection/
   - DBM + APM: https://docs.datadoghq.com/database_monitoring/connect_dbm_and_apm/?tab=go#before-you-begin

2. From tracer debug logs, confirm:
   - Are DB spans created at all (spans with `.query` etc.)?
   - Are they being sent to the Agent?

3. Check hidden span metadata:
   - If no `dbm_trace_injected` → tracer is not injecting DBM comments (propagation not configured, unsupported library, wrong version, etc.)

4. Obtain from the user:
   - Code snippet showing how the DB call is made
   - Tracer configuration / environment variables (`DD_DBM_PROPAGATION_MODE`, etc.)
   - Tracer debug logs around the time the query runs

**Escalation:**
- APM / #support-apm for span creation/propagation issues
- DBM if comments appear to be stripped by DB or Agent

---

### Bucket C: Sample Has Comments, but Trace is Missing or Not Correlated

**Scenario:**
- Question 1 = "Yes" (sample/plan exists)
- Question 2 = "Yes" (comments exist)
- Question 3 = "Trace missing / not found / not linked"

**Interpretation:**
At some point, a DB span existed and injected comments (DBM captured them), but:
- The trace/span was not retained (sampling, retention, ingestion), OR
- The query text/signatures do not match, so correlation cannot be resolved

**What to Check:**

1. Confirm APM ingestion and retention settings (intelligent retention, custom filters, etc.)
2. Confirm DBM Agent configuration:
   - Ensure `collect_comments` is not disabled in `obfuscator_options` for that DB
3. Check normalized vs exact statement matching:
   - Compare APM and DBM query text
   - Compare `_dd.query_signature` and `@trace.span.resource_hash` if present

**Escalation:**
- APM / #support-apm for ingestion/retention/sampling issues
- DBM / #support-database-monitoring if comments are collected but unused or mismatched

---

### Bucket D: Sample/Plan Exists, Span Exists, but No Correlation

**Scenario:**
- Question 1 = "Yes" (sample/plan exists)
- Question 2 = "No" (comments do not exist)
- Question 3 = "Trace with DB query exists"

**Interpretation:**
- If the setup qualifies as supported in public docs → investigate why comments are not being injected
- Common case: stored procedures/prepared statements (not tracked the same way across DBMSs)
- For OpenTelemetry: relies on Statement-based correlation

**Statement-Based Correlation:**
We are looking for the span's `resource_hash` to equal the DBM sample's `@db.query_signature`

**Common Fix (Agent 7.63+):**

Add to `datadog.yaml`:
```yaml
apm_config:
  sql_obfuscation_mode: "obfuscate_and_normalize"
```

**For SQL Server (Agent 7.74+):**

In `sqlserver.d/conf.yaml`:
```yaml
obfuscator_options:
  replace_bind_parameter: true
```
This replaces bind parameters (e.g. `@p1`) with `?` to match APM-side query.

**For OTel, check:**
- OTel intake paths / config
- Raw span metadata:
  - `span.type` ∈ ['sql', 'postgres', 'mysql', 'sql.query']
  - `span.service` is set to the calling service
  - `_dd.query_signature` or `@trace.span.resource_hash` matches a DBM query

**Escalation:** DBM / #support-database-monitoring (OTel mapping)

---

### Bucket E: "No database services found for this service" (APM service page)

**Typically means:**
There are no query metrics associated with this APM service, even if some samples correlate.

**What to Check:**
- For SQL Server: review `lookback_window` for query metrics (especially for infrequent queries)
- Otherwise, troubleshoot missing query metrics per DBM docs

**Escalation:** DBM / #support-database-monitoring

---

## Supported APM Integrations

| Tracing Library | Library/Framework | Min Version | APM Span Component |
|-----------------|-------------------|-------------|---------------------|
| Go (dd-trace-go) | database/sql | >= 1.44.0 | database/sql |
| Go (dd-trace-go) | sqlx | >= 1.44.0 | jmoiron/sqlx |
| Java (dd-trace-java) | jdbc | >= 1.11.0 | java-jdbc |
| Ruby (dd-trace-rb) | pg | >= 1.8.0 (1.12.0+ recommended) | pg |
| Ruby (dd-trace-rb) | mysql2 | >= 1.8.0 (1.12.0+ recommended) | mysql2 |
| Python (dd-trace-py) | psycopg2 | >= 1.9.0 | psycopg |
| .NET (dd-trace-dotnet) | Npgsql | >= 2.26.0 | Npgsql |
| .NET (dd-trace-dotnet) | MySql.Data | >= 2.26.0 | MySql |
| .NET (dd-trace-dotnet) | MySqlConnector | >= 2.26.0 | MySql |
| .NET (dd-trace-dotnet) | ADO.NET | >= 2.26.0 | SqlClient / AdoNet |
| PHP (dd-trace-php) | pdo | >= 0.86.0 | pdo |
| PHP (dd-trace-php) | MySQLi | >= 0.86.0 | mysqli |
| Node.js (dd-trace-js) | postgres | >= 3.17.0 | pg |
| Node.js (dd-trace-js) | mysql | >= 3.17.0 | mysql |
| Node.js (dd-trace-js) | mysql2 | >= 3.17.0 | mysql2 |

## Propagation Modes

| Setting | What Gets Injected as SQL Comments |
|---------|-----------------------------------|
| `full` | dddbs, dde, ddps, ddpv, traceparent (trace_id, span_id) |
| `service` | dddbs, dde, ddps, ddpv (no trace correlation) |
| `none` | Nothing injected |

**Environment variable:** `DD_DBM_PROPAGATION_MODE=full`

**Full mode is REQUIRED for APM→DBM correlation with SQL Server (.NET/Java)**

For SQL Server with .NET/Java:
- Agent version >= 7.55.0
- Java tracer >= 1.39.0
- .NET tracer >= 3.3.0
- Uses `SET CONTEXT_INFO` instead of SQL comments

## Escalation Summary

| Area | Typical Symptoms |
|------|------------------|
| APM | No DB spans in traces; spans not created; spans not ingested/retained; sampling/filtering; required tags missing |
| DBM | No query samples or explain plans; comments not captured or stripped; query metrics missing; "Upstream services" issues; OTel DBM mapping; "No database services found" |

## Useful Searches

### Find DBM samples with APM correlation data
```
@db.metadata.comments:* OR @trace.caller.service:*
```

### Check trace mode in DBM samples
```
@trace.mode:full
@trace.mode:service
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ddalexvea) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
