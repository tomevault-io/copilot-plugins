## xatu-cbt

> Xatu CBT contains the models and tests for the [xatu](https://github.com/ethpandaops/xatu) project. [CBT](https://github.com/ethpandaops/cbt) is a lightweight clickhouse only tool, similar to DBT/sqlmesh, that allows you to define models and run them against a clickhouse database to perform transformations.

# Xatu CBT

## Overview

Xatu CBT contains the models and tests for the [xatu](https://github.com/ethpandaops/xatu) project. [CBT](https://github.com/ethpandaops/cbt) is a lightweight clickhouse only tool, similar to DBT/sqlmesh, that allows you to define models and run them against a clickhouse database to perform transformations.

## Quirks

- Models are defined once but reused for multiple ethereum networks.
- We template `${NETWORK_NAME}` as the database in the [migrations](./migrations).
- Models must just reference tables in the same database, as we want to be able to reuse the models across many networks.
- Tests run against a network, not a specific fork.

## File Organisation

- `models/external` - contains the external models in a flat file structure where the file name is the table name.
- `models/transformations` - contains the transformations models in a flat file structure where the file name is the table name.
- `migrations` - database agnostic [go-migrate](https://github.com/golang-migrate/migrate) migrations that are run against the database.
- `tests/${network}/models` - contains the test definitions for models per network.

## Testing

The following structure is expected for each network:

```
tests/
├── ${network}/
│   └── models/                    # Test definitions
│       └── canonical_beacon_block.yaml
```

- Each network **MUST** have a file for every model being tested in the models directory.

### CLI

```bash
# Test all models for a network
xatu-cbt test all --network mainnet

# Test specific models
xatu-cbt test models fct_block --network mainnet
xatu-cbt test models fct_block,fct_attestation --network mainnet
```

## Models

### External Models

- Defines an external data source for transformations and provides SQL on how to get the current bounds of the data available as min/max integers.
- While we create views for each `default.table` to `$network.table` in the [migrations](./migrations), for performance reasons we do not use that for the external model definitions.
- External models should be filtered by the primary partition column for incremental scanning.
- For DateTime columns, we should use the `toUnixTimestamp` function to convert to a Unix timestamp integer as CBT expects integers for min/max bounds.

#### Example

```sql
---
table: beacon_api_eth_v1_events_block
cache:
  incremental_scan_interval: 5s
  full_scan_interval: 24h
---
SELECT
    toUnixTimestamp(min(slot_start_date_time)) as min,
    toUnixTimestamp(max(slot_start_date_time)) as max
-- Use the default database as predicate pushdown does not work with views.
-- This gives 2-3x the performance.
-- Once we move the data into the mainnet database, we no longer need this.
FROM `default`.`{{ .self.table }}`
WHERE
    meta_network_name = 'mainnet'
{{ if .cache.is_incremental_scan }}
    AND (
      slot_start_date_time <= fromUnixTimestamp({{ .cache.previous_min }})
      OR slot_start_date_time >= fromUnixTimestamp({{ .cache.previous_max }})
    )
{{ end }}
```

### Transformations Models

- Transformation models should use the `$network.external_table` view if using an external model as a dependency.
- Model naming conventions are documented in [NAMING_CONVENTIONS.md](./NAMING_CONVENTIONS.md)

---
> Source: [ethpandaops/xatu-cbt](https://github.com/ethpandaops/xatu-cbt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
