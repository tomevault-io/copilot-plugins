## evbf2-mcp-server

> MCP (Model Context Protocol) server for the Evonith Steel Blast Furnace #2.

# Evonith MCP Server — Developer Guide

MCP (Model Context Protocol) server for the Evonith Steel Blast Furnace #2.
Serves online PLC data and offline batch data from InfluxDB v3 Cloud Serverless over SSE/HTTP.

## Repository Layout

```
mcp_servers_evonith/
├── config/
│   └── measurements.yaml        # Schema registry: measurements, fields, descriptions
├── scripts/
│   └── setup.sh                 # Raspberry Pi deployment script
├── src/evonith_mcp/
│   ├── config.py                # pydantic-settings; reads .env
│   ├── server.py                # Starlette app, 17 registered MCP tools, bearer auth
│   ├── influx/
│   │   ├── client.py            # Two singleton clients (online + offline)
│   │   └── queries.py           # SQL query builders
│   ├── tools/
│   │   ├── online_data.py       # 7 tools: process params, temps, heat loads, etc.
│   │   ├── offline_data.py      # 6 tools: DPR, raw materials, hot metal/slag, charge
│   │   └── plot_gen.py          # 4 tools: trend, profile, scatter, shift comparison
│   └── utils/
│       ├── time_utils.py        # Parse "last 8h", ISO strings → (start, end) datetimes
│       └── sanitizer.py        # AST-based plot code safety checker
├── systemd/
│   └── evonith-mcp.service      # systemd unit for Raspberry Pi
├── tests/
│   ├── conftest.py              # Auto-skip integration tests without credentials
│   ├── test_time_utils.py       # Unit: time parsing
│   ├── test_sanitizer.py        # Unit: plot code safety
│   ├── test_queries.py          # Unit: SQL query builders
│   ├── test_plot_gen.py         # Unit: generated code runs and passes safety check
│   ├── test_online_data.py      # Integration: all online tools (@pytest.mark.integration)
│   └── test_offline_data.py     # Integration: all offline tools (@pytest.mark.integration)
├── .env                         # Credentials (gitignored)
├── .env.example                 # Template with placeholders only
└── pyproject.toml
```

## Development Setup

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
```

Copy `.env.example` to `.env` and fill in credentials. Never commit `.env`.

## Running the Server

```bash
# From repo root with .venv active
python -m evonith_mcp.server
# or via pyproject.toml entrypoint:
evonith-mcp
```

Health check: `curl http://localhost:8765/health`

## Running Tests

```bash
# Unit tests only (no InfluxDB needed)
pytest tests/ -m "not integration"

# Unit + integration (requires valid .env with InfluxDB credentials)
pytest tests/

# Single test file
pytest tests/test_queries.py -v
```

Integration tests are marked `@pytest.mark.integration` and auto-skipped when
`INFLUX_ONLINE_TOKEN` is absent from the environment.

## Key Architectural Decisions

### Two Separate InfluxDB Databases
- **Online** (`bf2_evonith_raw`): PLC data at 10-second cadence. Token: `INFLUX_ONLINE_TOKEN`.
- **Offline** (`bf2_evonith_offline_utc`): Batch Excel-parsed data ingested periodically. Token: `INFLUX_OFFLINE_TOKEN`.

Two separate `InfluxDBClient3` singletons in `client.py`, one per database.

### SELECT * for Raw Queries
InfluxDB v3 returns all columns regardless — listing fields is redundant for raw queries.
`build_raw_query()` always uses `SELECT *`. Explicit field lists are only needed for
`build_windowed_query()` because `AVG(field) AS field` must name each field.

### Windowed vs Raw Queries
All online tools support an optional `window_period` parameter (e.g. `"5m"`, `"1h"`).
- **Raw**: `SELECT * FROM measurement WHERE time >= ... AND time <= ... ORDER BY time`
- **Windowed**: `SELECT DATE_BIN(INTERVAL '...', time, ...) AS time, AVG(f) AS f, ... GROUP BY 1`

Strongly recommend `window_period` for online queries longer than 1 hour (10s cadence → large datasets).

### Timestamp Format
`timestamp '2026-03-20T00:00:00.000Z'` — millisecond precision, UTC literal.
Built by `to_influx_ts()` in `time_utils.py`.

### Offline Data Timestamp Caveat
The `time` column in offline measurements is the **ingestion timestamp**, not the actual
measurement/shift time. Data can be hours or days behind actual events. Always use
`get_latest_offline_reading` when you need "most recent available" regardless of lag.

### Schema Registry (`config/measurements.yaml`)
Single source of truth for:
- Measurement → field name mappings (verified against live InfluxDB)
- Field descriptions for LLM context
- Material prefixes for `rm_updated_data` pandas-side column filtering
- Height-level to field-name mappings for `temperature_profile`

Used by `list_online_tags`, `list_offline_datasets`, and windowed query field lists.
Do not guess field names — verify against the YAML or live data.

### Plot Code Safety
All generated plot code passes through `validate_plot_code()` (AST-based) before being returned.
- Allowed imports: `plotly`, `pandas`, `numpy` only
- Forbidden calls: `exec`, `eval`, `open`, `compile`, `__import__`
- Generated code contract: receives `df` (DataFrame), produces `fig` (Plotly figure)
- Streamlit calls: `exec(code, {"df": df})` then renders `fig`

User-supplied strings embedded in code are escaped via `_safe_str()` in `plot_gen.py`.

### Authentication
Bearer token middleware (`BearerAuthMiddleware`) validates `Authorization: Bearer <MCP_API_KEY>`
on all requests except `GET /health`. Set a strong random key in `.env`.

## Adding a New Tool

1. Add the implementation function in the appropriate `tools/` module.
2. Register a `Tool(...)` entry in `TOOLS` list in `server.py`.
3. Add a dispatch entry in `_dispatch()` in `server.py`.
4. Add integration tests in the corresponding `tests/test_*.py` file.
5. If the tool uses a new measurement, add it to `config/measurements.yaml`.

## Raspberry Pi Deployment

```bash
# On the Pi, from repo root:
bash scripts/setup.sh

# Service management:
sudo systemctl status evonith-mcp
sudo systemctl restart evonith-mcp
sudo journalctl -u evonith-mcp -f
```

The systemd unit reads credentials from `.env` via `EnvironmentFile=`.
Server listens on `MCP_HOST:MCP_PORT` (default `0.0.0.0:8765`).

The Streamlit Cloud webapp connects to `http://<pi-static-ip>:8765/sse` with
`Authorization: Bearer <MCP_API_KEY>`.

## Environment Variables

| Variable | Description |
|---|---|
| `INFLUX_HOST` | InfluxDB v3 Cloud endpoint URL |
| `INFLUX_ORG` | InfluxDB org name |
| `INFLUX_ONLINE_TOKEN` | Token for `bf2_evonith_raw` |
| `INFLUX_OFFLINE_TOKEN` | Token for `bf2_evonith_offline_utc` |
| `INFLUX_DATABASE` | Online database name |
| `INFLUX_OFFLINE_DATABASE` | Offline database name |
| `MCP_HOST` | Bind address (default `0.0.0.0`) |
| `MCP_PORT` | Port (default `8765`) |
| `MCP_API_KEY` | Bearer token for client auth |
| `MEASUREMENTS_CONFIG` | Path to `measurements.yaml` (default `config/measurements.yaml`) |
| `INFLUX_CERT_PATH` | Optional TLS cert for custom CA |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saikumar-voptimai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/saikumar-voptimai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
