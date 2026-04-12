## analitiq-engine

> Analitiq Stream is a fault-tolerant data streaming framework for Python 3.11+ enabling reliable data movement between sources and destinations. Uses Poetry for dependency management with async/await streaming.

# CLAUDE.md

## Project Overview

Analitiq Stream is a fault-tolerant data streaming framework for Python 3.11+ enabling reliable data movement between sources and destinations. Uses Poetry for dependency management with async/await streaming.

## Development Commands

```bash
# Setup
poetry install                              # Install dependencies
poetry install -E "kafka analytics"         # With extras
poetry shell                                # Activate venv

# Testing
poetry run pytest                           # All tests
poetry run pytest --cov=src --cov-report=html
poetry run pytest -m "unit"                 # By marker: unit, integration, slow

# Code Quality
poetry run black src/ && poetry run isort src/  # Format
poetry run mypy src/                            # Type check
poetry run flake8 src/                          # Lint
poetry run pre-commit run --all-files           # All hooks
```

## Project Structure

```
src/
├── shared/                      # Shared utilities used by source and destination
│   ├── database_utils.py        # SSL mode, identifiers, table names
│   └── rate_limiter.py          # Token bucket rate limiter
│
├── source/                      # Source connectors (read operations)
│   ├── connectors/              # Protocol implementations
│   │   ├── base.py              # BaseConnector ABC
│   │   ├── api.py               # APIConnector
│   │   └── database.py          # DatabaseConnector (uses drivers)
│   └── drivers/                 # Database-specific drivers
│       ├── base.py              # BaseDatabaseDriver ABC
│       ├── postgresql.py        # PostgreSQLDriver
│       └── factory.py           # Driver factory
│
├── destination/                 # Destination components (write operations)
│   ├── base_handler.py          # BaseDestinationHandler ABC
│   ├── server.py                # gRPC server implementation
│   ├── schema_contract.py       # Arrow-based type casting (DestinationSchemaContract)
│   ├── connectors/              # Destination handlers (renamed from handlers/)
│   │   ├── database.py          # DatabaseDestinationHandler (SQLAlchemy + Arrow)
│   │   ├── api.py               # ApiDestinationHandler
│   │   ├── file.py              # FileDestinationHandler
│   │   └── stream.py            # StreamDestinationHandler
│   ├── formatters/              # JSONL, CSV, Parquet formatters
│   ├── storage/                 # Local storage backends
│   └── idempotency/             # ManifestTracker for file-based idempotency
│
├── engine/                      # Core engine (renamed from core/)
│   ├── engine.py                # StreamingEngine - ETL pipeline processor
│   ├── pipeline.py              # Pipeline configuration interface
│   ├── pipeline_config_prep.py  # Multi-environment config loading
│   ├── data_transformer.py      # Field transformations
│   └── credentials.py           # Credentials management
│
├── state/                       # Fault tolerance (renamed from fault_tolerance/)
│   ├── state_manager.py         # State management
│   ├── state_storage.py         # Local state storage
│   ├── retry_handler.py         # Retry handling
│   ├── circuit_breaker.py       # Circuit breaker pattern
│   ├── dead_letter_queue.py     # Dead letter queue
│   └── log_storage.py           # Log storage
│
├── grpc/                        # gRPC client and generated code
├── models/                      # Pydantic models
├── config/                      # Configuration loading
├── secrets/                     # Secret resolvers
├── schema/                      # Schema drift detection
├── transformations/             # Transformation registry
├── main.py                      # Entry point (dual-mode: engine/destination)
└── runner.py                    # PipelineRunner
```

### Import Path Changes

| Old Import | New Import |
|------------|------------|
| `from src.connectors` | `from src.source.connectors` |
| `from src.core` | `from src.engine` |
| `from src.fault_tolerance` | `from src.state` |
| `from src.destination.handlers` | `from src.destination.connectors` |

Backward compatibility aliases exist but will emit deprecation warnings.

## Configuration Architecture

Configuration is assembled at runtime from modular files. Paths are defined in `analitiq.yaml`.
Pipeline discovery uses `manifest.json` as the central index; only `active` pipelines are executable.

```
project_root/
├── connectors/{slug}/              # Downloaded from GitHub (one repo per connector)
│   └── definition/
│       ├── connector.json          # Connector metadata, auth, form_fields
│       ├── manifest.json           # Lists public endpoints
│       └── endpoints/              # Public endpoints (API schemas)
│           └── {name}.json
├── connections/{alias}/            # User-created, human-readable alias
│   ├── connection.json             # Host, connector_slug, parameters
│   ├── .secrets/
│   │   └── credentials.json        # Secret key-value pairs
│   └── endpoints/                  # Private endpoints (DB table schemas)
│       └── {name}.json
├── pipelines/
│   ├── manifest.json               # Central index of all pipelines
│   └── {pipeline_id}/              # Pipeline directory (name = pipeline ID)
│       ├── pipeline.json           # Pipeline config (connections, runtime, stream IDs)
│       └── streams/
│           └── {stream_id}.json    # Individual stream config files
└── analitiq.yaml                   # Path configuration
```

### Endpoint References

Streams reference endpoints using scoped path format:
- `"connector:{slug}/{name}"` → public endpoint from a connector
- `"connection:{alias}/{name}"` → private endpoint from a connection

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `PIPELINE_ID` | Yes | Pipeline ID (matches `pipeline_id` in manifest.json) |

### Using PipelineConfigPrep

```python
from src.engine.pipeline_config_prep import PipelineConfigPrep

# Requires PIPELINE_ID env var. Reads paths from analitiq.yaml.
# Discovers pipeline via manifest.json, enforces active status.
prep = PipelineConfigPrep()
pipeline_config, stream_configs, connections, endpoints, connectors = prep.create_config()
```

## Configuration Schemas

### Manifest (`pipelines/manifest.json`)

Central index of all pipelines. Only pipelines with `status: "active"` can be executed.

```json
{
  "pipelines": [
    {
      "pipeline_id": "mysql-to-postgresql",
      "name": "MySQL to PostgreSQL",
      "path": "mysql-to-postgresql/pipeline.json",
      "status": "active",
      "schedule": { "type": "manual", "cron": null },
      "streams": ["stream-1"]
    }
  ]
}
```

### Pipeline (`pipelines/{id}/pipeline.json`)

Lightweight reference documents. Connections are aliases, stream IDs reference separate stream files.

```json
{
  "pipeline": {
    "name": "Pipeline Name",
    "status": "active",
    "version": 1,
    "connections": {
      "source": "my-api-connection",
      "destinations": ["prod-postgres"]
    },
    "streams": ["stream-1"],
    "schedule": { "type": "interval", "timezone": "UTC", "interval_minutes": 1440 },
    "engine": { "vcpu": 1, "memory": 8192 },
    "runtime": {
      "buffer_size": 5000,
      "batching": { "batch_size": 100, "max_concurrent_batches": 3 },
      "logging": { "log_level": "INFO", "metrics_enabled": true },
      "error_handling": { "strategy": "dlq", "max_retries": 3, "retry_delay": 5 }
    }
  },
  "streams": []
}
```

### Stream (`pipelines/{id}/streams/{stream_id}.json`)

Individual stream configuration with source, destinations, and mapping.

```json
{
  "stream_id": "stream-1",
  "is_enabled": true,
  "source": {
    "connection_ref": "my-api-connection",
    "endpoint_ref": "connector:wise/transfers",
    "primary_key": ["id"],
    "replication": { "method": "incremental", "cursor_field": ["created"] }
  },
  "destinations": [{
    "connection_ref": "prod-postgres",
    "endpoint_ref": "connection:prod-postgres/public_transfers",
    "write": { "mode": "upsert", "conflict_keys": ["id"] }
  }],
  "mapping": { "assignments": [] }
}
```

### Connectors (`connectors/{slug}/definition/connector.json`)

Downloaded from GitHub. Define connector type, auth, form fields, rate limits.

```json
{
  "connector_id": "uuid",
  "connector_name": "Wise",
  "connector_type": "api",
  "slug": "wise",
  "base_url": "https://api.transferwise.com",
  "auth": { "type": "api_key" },
  "headers": { "Authorization": "Bearer ${access_token}" }
}
```

Connector types: `api`, `database`, `file`, `stdout`

### Connections (`connections/{alias}/connection.json`)

User-created. Reference a connector by `connector_slug`. Credentials use `${VAR}` placeholders resolved from `connections/{alias}/.secrets/credentials.json`.

**Database connection:**
```json
{
  "connector_slug": "postgresql",
  "host": "${DB_HOST}",
  "parameters": {
    "database": "${DB_NAME}",
    "username": "${DB_USER}",
    "password": "${DB_PASSWORD}",
    "port": 5432
  }
}
```

**API connection:**
```json
{
  "connector_slug": "wise",
  "host": "https://api.transferwise.com",
  "parameters": {
    "headers": { "Authorization": "Bearer ${access_token}" }
  }
}
```

## Storage Backends

All storage (state, logs, DLQ, metrics) uses local filesystem:
- **Local:** `{type}/{pipeline_id}/`

Centralized directories at project root: `state/`, `logs/`, `deadletter/`, `metrics/`

## gRPC Streaming Architecture

Decoupled destination services via bidirectional gRPC streaming.

```
ENGINE CONTAINER                    DESTINATION CONTAINER
Extract -> Transform -> GrpcLoad   DestinationGRPCServer -> Handler
```

**Dual-mode Docker:** Same image runs as engine or destination via `RUN_MODE` env var. Both containers use `PipelineConfigPrep` to load configuration from the same `PIPELINE_ID`.

| Engine Variable | Description |
|-----------------|-------------|
| `DESTINATION_GRPC_HOST` | Enables gRPC mode |
| `DESTINATION_GRPC_PORT` | Default: `50051` |
| `GRPC_TIMEOUT_SECONDS` | Default: `300` |

| Destination Variable | Description |
|---------------------|-------------|
| `GRPC_PORT` | Default: `50051` |
| `DESTINATION_INDEX` | Which destination from pipeline config (default: `0`) |

**Configuration Loading:** Both engine and destination load configuration via `PipelineConfigPrep` using the same `PIPELINE_ID`. The destination uses `DESTINATION_INDEX` to select which destination connection from the pipeline's `connections.destinations` list. Credentials are loaded locally from the config volume, never sent over gRPC.

**Protocol:** Opaque cursors, batch idempotency via `(run_id, stream_id, batch_seq)`, all-or-nothing batches.

## Destination Architecture

Configuration-driven destination system with layered abstractions. Single Docker image handles all destination types based on `connector_type` defined in the connector (looked up via `connector_id` on the connection).

**Handlers:** `db` (SQLAlchemy), `postgresql` (legacy asyncpg), `api`, `file`, `stdout`

**Formatters:** `jsonl`, `csv`, `parquet`

**Storage:** `local`

**Idempotency:** Database handlers use `_batch_commits` table; file handlers use `_manifest.json`

## Arrow-Based Type System

Type casting uses PyArrow for efficient columnar conversion instead of row-by-row Python coercion.

**Architecture:**
```
Source (native types) -> Arrow Table (canonical schema) -> Destination (native types)
```

**Key Components:**
- `DestinationSchemaContract` (`src/destination/schema_contract.py`): Builds Arrow schema from endpoint definition, provides vectorized batch casting
- `target.type`: Generic type for Pydantic validation (`integer`, `string`, `datetime`, etc.)
- `target.dest_type`: Native SQL type for DDL/casting (`BIGINT`, `VARCHAR(50)`, `DATETIME`, etc.)

**Type Mapping Flow:**
1. `source_to_generic` mapping provides generic types from source fields
2. `generic_to_destination` mapping provides native destination types per connection
3. `_normalize_mapping_config()` sets both `target.type` and `target.dest_type` on assignments
4. `DestinationSchemaContract` builds Arrow schema from endpoint columns
5. `prepare_records()` performs vectorized casting using PyArrow

**Supported Native Types:**
- Integer: `BIGINT`, `INTEGER`, `SMALLINT`, `TINYINT`
- Float: `FLOAT`, `DOUBLE`, `REAL`, `DECIMAL(p,s)`
- String: `VARCHAR(n)`, `TEXT`, `CHAR`
- Timestamp: `TIMESTAMP`, `TIMESTAMPTZ`, `DATETIME`
- Date/Time: `DATE`, `TIME`
- Boolean: `BOOLEAN`, `BOOL`
- JSON: `JSON`, `JSONB` (stored as string)
- Binary: `BYTEA`, `BLOB`

## Incremental Replication

- **`replication_key`**: Field for ordering/tracking sync progress (e.g., `updatedAt`)
- **`safety_window_seconds`**: Offset subtracted from cursor (60-300s typical) to handle clock skew
- **`bookmarks`**: Array of `{partition, cursor, aux}` for checkpoint tracking
- Engine uses inclusive mode (`>=`) for cursor comparison; duplicates handled via upsert

## Testing

### Running docker test locally
When running docker test locally:
 - pass the required env vars to the container.
 - examine docker logs for errors.

Example:
```shell
cd docker && \
   PIPELINE_ID=my-pipeline-id \
   docker compose run --rm source_engine 2>&1
```

## Data Flow

1. **Extract**: Read from source in batches
2. **Transform**: Apply field mappings and transformations
3. **Load**: Write to destination with fault tolerance
4. **Checkpoint**: Save progress state

## Key Design Patterns

- Async/await for all I/O
- Producer-consumer via async queues
- Factory pattern for connectors and handlers
- Registry pattern for destination handlers, formatters, storage backends
- Composition pattern for FileDestinationHandler (formatter + storage + manifest)
- Driver delegation for databases
- Sharded state checkpoints
- Pydantic V2 validation

## Coding Guidelines

- Python 3.11+ with Pydantic V2 validation
- No backward compatibility code
- No emojis in code
- Modular, pythonic code with explicit error paths
- Use ExceptionGroup and except* where helpful
- Deterministic behavior, idempotent writes, safe retries
- Update README.md when modifying documented functionality
- DO NOT ADD LEGACY SUPPORT OR BACKWARD COMPATIBILITY, UNLESS EXPLICITLY INSTRUCTED!


## Working on Issues:

1. Examine GitHub issues, pick the easist one to implement.
2. Create a new branch for the issue.
3. Implement the issue.
4. Commit your changes and push to the branch.
5. Create a pull request (if not yet created).
6. Run the PR Review Process
7. Wait for feedback from the review executor.

## PR Review Process:

1. Use `/pr-review-toolkit` to review the PR after you have implemented all changes.
2. Wait for feedback from the review executor.
3. Determine if the raised issues are legitimate or not.
   a. if the issue is legitimate and relevant to the PR, fix it.
   b. if the issue is outside the scope of the PR, check if there is a related issue in the GitHub issue tracker. If not, create a new issue in GitHub and move on.
   c. If the issue is not a legitimate problem, summarize your thoughts on the point and move on.
4. Once you fixed all issues that need fixing, commit fixes, push to the branch.
5. Use `/pr-review-toolkit` to review again
6. Continue doing this cycle until the PR is approved by the review executor.
7. Once the PR is approved, run the tests to make sure they all pass.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/analitiq-ai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/analitiq-ai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
