## automated-intelligence

> Interactive Tables configuration, latency constraints, and validation patterns


# Interactive Tables

This component validates Snowflake Interactive Tables — always-on, low-latency tables designed for sub-100ms point lookups.

## Architecture

```
RAW tables (customers, orders)
        │
        ▼
Interactive Tables (always-on, <100ms reads)
        │
        ├── INTERACTIVE.CUSTOMER_ORDER_ANALYTICS  (CLUSTER BY customer_id)
        └── INTERACTIVE.ORDER_LOOKUP              (CLUSTER BY order_id)
        │
        ▼
Validation tests (test_interactive_layer.sql — 10 tests)
```

## Key Constraints (Official Docs, 2025)

| Feature | Constraint |
|---------|-----------|
| **SELECT timeout** | 5 seconds maximum — **cannot be increased** |
| **DML** | INSERT OVERWRITE only (no UPDATE/DELETE/MERGE) |
| **CLUSTER BY** | Required on every Interactive Table |
| **Billing** | Always-on compute — no auto-suspend, no auto-scaling |
| **Warehouse** | Dedicated warehouse type (`AUTOMATED_INTELLIGENCE_INTERACTIVE_WH`) |
| **Use case** | Sub-second point lookups, NOT analytical queries |

### Critical: 5-Second Timeout

Interactive Tables enforce a hard 5-second SELECT timeout. Any query exceeding this limit is automatically cancelled. This means:
- Only simple, well-indexed point lookups are appropriate
- Avoid JOINs, aggregations, or full-table scans
- CLUSTER BY must align with your primary lookup pattern

## Table Definitions

Tables are created in root `setup.sql` (not in this directory) using CTAS pattern:

```sql
-- setup.sql:831
CREATE OR REPLACE INTERACTIVE TABLE INTERACTIVE.CUSTOMER_ORDER_ANALYTICS
  CLUSTER BY (customer_id)
AS
  SELECT ... FROM RAW.CUSTOMERS c JOIN RAW.ORDERS o ON ...;

-- setup.sql:849
CREATE OR REPLACE INTERACTIVE TABLE INTERACTIVE.ORDER_LOOKUP
  CLUSTER BY (order_id)
AS
  SELECT ... FROM RAW.ORDERS;
```

## Data Flow

Interactive Tables in this demo source from **RAW** tables directly:

```
RAW tables (customers, orders)
  → Interactive Tables (INTERACTIVE schema)
```

The `manual_refresh.sql` file also references Dynamic Tables for manual refresh operations.

## Validation Tests

`test_interactive_layer.sql` runs 10 tests with latency thresholds:

| Test | Threshold | What It Checks |
|------|-----------|----------------|
| Test 1: Row count validation | N/A | Counts match source tables |
| Test 2: Schema validation | N/A | Column types match expectations |
| Test 3: Data freshness | N/A | Verifies data freshness vs source |
| Test 4: Customer lookup by ID | <100ms | Point lookup on CLUSTER BY key |
| Test 5: Order lookup by ID | <50ms | Point lookup on CLUSTER BY key |
| Test 6: Recent orders | <200ms | Date-range filter |
| Test 7: Aggregation query | <300ms | Aggregation performance |
| Test 8-10: Additional tests | Various | Edge cases and concurrent access |

## Known Issues

1. **`setup_interactive.sql` referenced but missing** — Referenced in `test_interactive_layer.sql` and `README.md` (3 places) but does not exist; the actual DDL is in root `setup.sql`
2. **No auto-suspend** — The interactive warehouse runs continuously; remember to manually suspend when not demoing
3. **Always-on billing** — Interactive Tables bill for always-on compute even when idle
4. **README references `demo_west_coast_manager.ipynb`** — this notebook does not exist

## Additional Files

- `test_interactive_layer.sql` — 10 validation tests
- `manual_refresh.sql` — Manual refresh commands for interactive and dynamic tables
- `realtime_demo.py` — Python demo script for real-time latency testing
- `README.md` — Documentation

## Running Validation

```sql
USE WAREHOUSE AUTOMATED_INTELLIGENCE_INTERACTIVE_WH;
USE SCHEMA AUTOMATED_INTELLIGENCE.INTERACTIVE;

-- Quick latency check
SELECT * FROM CUSTOMER_ORDER_ANALYTICS WHERE CUSTOMER_ID = 1;
SELECT * FROM ORDER_LOOKUP WHERE ORDER_ID = '<uuid>';
```

## When NOT to Use Interactive Tables

- Analytical queries with aggregations → Use Dynamic Tables or standard tables
- Queries that may exceed 5 seconds → Use standard warehouses
- Cost-sensitive workloads → Always-on billing adds up quickly
- Tables needing UPDATE/DELETE → Use standard tables with MERGE

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iamontheinet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
