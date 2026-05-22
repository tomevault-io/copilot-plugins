## metabase-flightsql-driver

> This document captures key learnings, patterns, and best practices for developing and maintaining the Metabase Arrow Flight SQL driver.

# CLAUDE.md - Development Guide for Metabase Arrow Flight SQL Driver

This document captures key learnings, patterns, and best practices for developing and maintaining the Metabase Arrow Flight SQL driver.

## Project Overview

This is a Clojure-based Metabase driver that enables connections to databases using the Apache Arrow Flight SQL JDBC driver. Primary use cases include connecting to Spice.ai OSS and GizmoSQL.

## Project Structure

```
metabase-flightsql-driver/
├── src/metabase/driver/arrow_flight_sql.clj  # Main driver code
├── resources/metabase-plugin.yaml             # Plugin manifest
├── scripts/metabase_setup.py                  # Automated setup & testing
├── docker-compose.yaml                        # Container orchestration
├── gizmosql/init.sql                          # GizmoSQL test data
├── spice/spicepod.yaml                        # Spice.ai configuration
└── data/                                      # Parquet files for Spice
```

## Development Commands

```bash
# Start all services (builds driver automatically)
podman compose up --build

# Rebuild only the driver
podman compose down metabase builder && podman compose up -d builder

# Start Metabase after rebuild
podman compose up -d metabase

# Check Metabase logs
podman logs metabase 2>&1 | tail -100

# Run automated setup script
python scripts/metabase_setup.py
```

## Key Driver Implementation Details

### Connection Pooling Fix

**Critical**: The `connection-details->spec` method must return a stable hash. Anonymous functions in the return map cause Metabase to constantly invalidate the connection pool.

```clojure
;; BAD - Creates new function on every call, breaks connection pooling
{:classname "..."
 :subprotocol "..."
 :cast (fn [col val] ...)}  ; Anonymous fn = different hash each time

;; GOOD - Use a named function defined once
(defn- arrow-flight-sql-cast-fn [col val] ...)

{:classname "..."
 :subprotocol "..."
 :cast arrow-flight-sql-cast-fn}  ; Stable reference
```

**Symptoms of broken pooling:**
- Logs show: `Hash of database X details changed; marking pool invalid`
- Connection count shows: `arrow-flight-sql DB X connections: 0/0`
- Random query failures on dashboard load

### Transaction Isolation Override

Arrow Flight SQL servers may not support transaction isolation level checks. The driver overrides `do-with-connection-with-options` to skip these checks:

```clojure
(defmethod sql-jdbc.execute/do-with-connection-with-options :arrow-flight-sql
  [driver db-or-id-or-spec options f]
  ;; Skip set-best-transaction-level! which causes NPE with Flight SQL
  ...)
```

## Metabase API Patterns

### Field Filters with Table Aliases

When using field filters in native SQL with JOINs, you must specify the table alias in the template tag:

```python
# Template tag with alias for "orders o" table
{
    "status": {
        "id": "status",
        "name": "status",
        "display-name": "Order Status",
        "type": "dimension",
        "dimension": ["field", field_id, None],
        "widget-type": "category",
        "alias": "o.status"  # Critical for JOINs!
    }
}
```

**SQL syntax for field filters:**
```sql
-- Single table (no alias needed)
SELECT * FROM orders [[WHERE {{status}}]]

-- With JOINs (alias required in template tag)
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE 1=1 [[AND {{status}}]] [[AND {{country}}]]
```

### Dashboard Parameter Mappings

Connect dashboard filters to card template tags:

```python
parameter_mappings = [{
    "parameter_id": "status",       # Dashboard parameter ID
    "card_id": card_id,
    "target": ["dimension", ["template-tag", "status"]]
}]
```

### Dashboard Parameters

```python
dashboard_params = [{
    "id": "status",
    "name": "Order Status",
    "slug": "status",
    "type": "string/=",      # For text fields
    "sectionId": "string"
}, {
    "id": "date_range",
    "name": "Date Range",
    "slug": "date_range",
    "type": "date/all-options",  # For date fields
    "sectionId": "date"
}]
```

## Test Data Setup (GizmoSQL)

The `gizmosql/init.sql` creates test schemas:
- `sales`: orders, customers, products, order_items
- `hr`: employees, departments, time_off_requests
- `analytics`: daily_metrics, website_events, campaign_performance

## Common Issues & Solutions

### Issue: Charts fail randomly on dashboard load
**Cause**: Connection pool invalidation (see Connection Pooling Fix above)
**Solution**: Ensure `connection-details->spec` returns stable values

### Issue: Field filter error "while preparing SQL"
**Cause**: Missing table alias in template tag for JOIN queries
**Solution**: Add `"alias": "table_alias.column"` to template tag

### Issue: "Prepared statement not found" errors
**Cause**: Concurrent query execution with unstable connections
**Solution**: Fix connection pooling, ensure pool maintains connections

### Issue: Charts display in question view but not dashboard
**Cause**: Missing `visualization_settings` on dashcards
**Solution**: Copy card's visualization_settings to dashcard when adding to dashboard

## Useful Metabase API Endpoints

```bash
# Health check
curl http://localhost:3000/api/health

# List databases
curl -H "x-api-key: $KEY" http://localhost:3000/api/database

# Get database metadata (tables/fields)
curl -H "x-api-key: $KEY" http://localhost:3000/api/database/{id}/metadata

# Get card details
curl -H "x-api-key: $KEY" http://localhost:3000/api/card/{id}

# Run card query with parameters
curl -X POST -H "x-api-key: $KEY" -H "Content-Type: application/json" \
  http://localhost:3000/api/card/{id}/query \
  -d '{"parameters":[{"type":"string/=","value":["Delivered"],"target":["dimension",["template-tag","status"]]}]}'

# Get dashboard
curl -H "x-api-key: $KEY" http://localhost:3000/api/dashboard/{id}
```

## External Resources

- [Metabase Field Filters Documentation](https://www.metabase.com/docs/latest/questions/native-editor/field-filters)
- [Metabase API Documentation](https://www.metabase.com/docs/latest/api)
- [Arrow Flight SQL JDBC Driver](https://arrow.apache.org/docs/java/flight_sql_jdbc_driver.html)
- [Spice.ai Documentation](https://spiceai.org/docs)
- [GizmoSQL (DuckDB-based)](https://github.com/gizmodata/gizmosql)

## Metabase Driver Development References

When implementing new driver features, reference these existing Metabase drivers:
- `metabase/driver/postgres.clj` - Full-featured SQL driver
- `metabase/driver/sql_jdbc/` - SQL JDBC base implementations
- `metabase/driver/sql/query_processor.clj` - Query processing methods

## Environment Variables

The `.env` file is **auto-generated** by the setup script - no manual creation needed.

```bash
METABASE_API_KEY=mb_...  # Generated by setup script, stored in .env
```

**Note**: The `.env` file is not required for `podman compose up` to work. It's only created after running the setup script and is used for subsequent API calls.

## Docker Services

| Service | Port | Description |
|---------|------|-------------|
| metabase | 3000 | Metabase UI |
| postgres | 5432 | Metabase app database |
| spiced | 50051, 8090, 9090 | Spice.ai Flight SQL |
| gizmosql | 31337 | GizmoSQL Flight SQL |

## Testing Checklist

- [ ] Single table queries work
- [ ] JOIN queries work
- [ ] Field filters work (single table)
- [ ] Field filters work with aliases (JOINs)
- [ ] Date filters work
- [ ] Dashboard loads all charts consistently
- [ ] Filters are interactive in dashboard
- [ ] Database sync completes successfully
- [ ] Connection test passes

---
> Source: [J0hnG4lt/metabase-flightsql-driver](https://github.com/J0hnG4lt/metabase-flightsql-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
