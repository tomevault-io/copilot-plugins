## th-strava-sensor

> This file provides guidance to AI coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

Strava Sensor is a Python tool that extracts device battery information from sports activity FIT files and publishes them as Home Assistant sensors via MQTT. It supports multiple sources for reading FIT files and uses a modular, pluggable architecture.

## Development Setup

### Essential Commands

```bash
# Install dependencies
pip install uv
uv sync

# Install pre-commit hooks
uv run pre-commit install

# Run all quality checks (linting, type checking, tests)
uv run ./scripts/run-all-checks.sh

# Run tests only
uv run pytest tests/ -v

# Run with coverage
uv run pytest tests/ --cov --cov-report=term-missing

# Code formatting and linting
uv run ruff check --fix
uv run ruff format

# Type checking
uv run pyright
```

### Running the Application

```bash
# Parse local FIT file
uv run parse-activity file:///path/to/activity.fit

# Parse from Garmin Connect
uv run parse-activity garmin://123456789

# Parse via Strava API (finds activity then downloads from other sources)
uv run parse-activity https://www.strava.com/activities/123456789

# Parse and publish to MQTT/Home Assistant
uv run parse-activity --publish file:///path/to/activity.fit

# Start webhook server for automatic Strava activity processing
uv run strava-sensor
```

## Architecture Overview

### Core Components

**Source System (`src/strava_sensor/source/`)**
- `base.py`: Abstract base class defining source interface with URI matching
- `file.py`: Local filesystem FIT file handling (`file://` scheme)
- `garmin.py`: Garmin Connect integration (`garmin://` scheme, Garmin Connect URLs)
- `strava.py`: Strava API integration with delegation to downstream sources

**FIT File Processing (`src/strava_sensor/fitfile/`)**
- `fitfile.py`: Garmin FIT SDK wrapper with error handling and device extraction
- `model.py`: Pydantic v2 models for device status with MQTT publishing capabilities

**Entry Points**
- `cli.py`: Main CLI interface with source initialization and coordination
- `webhook_server.py`: FastAPI server for Strava webhook processing with persistent MQTT client

**Web UI (`src/strava_sensor/ui/`)**
- `status_page.py`: NiceGUI status dashboard mounted into the FastAPI app
- The UI is registered from `webhook_server.py` via `register_status_page(app, ...)`
- UI actions should delegate to backend callables (e.g., MQTT reconnect/disconnect, manual FIT upload) instead of duplicating processing logic in UI handlers

**MQTT Integration (`src/strava_sensor/mqtt/`)**
- `mqtt.py`: MQTT client wrapper with Home Assistant auto-discovery support

### Key Design Patterns

1. **Source Delegation**: Strava source delegates to downstream sources (Garmin, File) since Strava doesn't provide direct FIT file access
2. **Environment-Driven Configuration**: Features enabled based on environment variables (optional sources, MQTT publishing)
3. **URI-Based Routing**: Sources match URIs via schemes and hostnames
4. **Graceful Error Handling**: Continue processing other devices if one fails, with typed exceptions

### Source Initialization Order

Sources are initialized in specific order in `cli.py:initialize_sources()`:
1. File source (always available)
2. Garmin source (if credentials available)
3. Strava source (if token available, receives all downstream sources for delegation)

This ordering enables Strava→Garmin→File delegation chain.

## Environment Variables

### Required for Garmin Integration
- `GARMIN_USERNAME`: Garmin Connect username
- `GARMIN_PASSWORD`: Garmin Connect password

### Required for Strava Integration
- `STRAVA_REFRESH_TOKEN`: Strava API refresh token
- `STRAVA_CLIENT_ID`: Required for webhook subscription
- `STRAVA_CLIENT_SECRET`: Required for webhook subscription
- `STRAVA_VERIFY_TOKEN`: Webhook verification token (optional, auto-generated if omitted)
- `STRAVA_WEBHOOK_URL`: Public HTTPS URL for webhook callbacks

### Required for MQTT Publishing
- `MQTT_BROKER_URL`: MQTT broker URL (e.g., `mqtt://broker:1883`)
- `MQTT_USERNAME`: MQTT username
- `MQTT_PASSWORD`: MQTT password

### Optional
- `WEBHOOK_PORT`: Port for webhook server (default: 8000)
- `GARMINTOKENS`: Custom path for Garmin token storage

## Development Guidelines

### Code Quality
- Line length: 100 characters
- Quote style: Single quotes
- Python version: 3.13+
- Type hints required for all public APIs
- Use Pydantic v2 for data validation
- Python import guidelines:
  - Prefer fully qualified external imports (e.g., `import datetime` instead of
    `from datetime import datetime`).
  - Abbreviate common modules on import:
    - `import typing as t`
    - `import pulumi as p`
    - `import pulumi_aws as aws`
    - `import pulumi_kubernetes as k8s`
    - `import collections.abc as c`
  - Do not use `from __future__ import annotations`.

### Error Handling
- Use typed exceptions from `fitfile.py`: `NotAFitFileError`, `CorruptedFitFileError`, `InvalidActivityFileError`
- Log errors with sufficient context
- Provide fallback mechanisms where recovery is possible

### Adding New Sources
1. Inherit from `BaseSource` in `source/base.py`
2. Define `uri_scheme` and/or `http_hosts` class attributes
3. Implement `read_activity()` method returning raw FIT bytes
4. Optional: Implement `find_activity()` for Strava delegation support
5. Add to `initialize_sources()` in `cli.py` (before Strava if supporting delegation)

### Device Model Overrides
Edit `MODEL_OVERRIDE` dictionary in `fitfile/model.py` to add manufacturer-specific device name mappings:
```python
MODEL_OVERRIDE = {
    'manufacturer_name': {
        'product_code': 'friendly_device_name'
    }
}
```

### Testing
- Use pytest with fixtures in `tests/fixtures/`
- Mock external dependencies (APIs, files)
- Maintain >90% test coverage
- Include corrupted file test cases

### MQTT Publishing
- Home Assistant auto-discovery topics: `homeassistant/device/strava-<serial>/config`
- Device state topics: `strava/<serial_number>/status`
- Published data includes battery level, status, and voltage sensors

## Common Workflows

### Processing Flow
1. CLI receives URI → source matching → activity data retrieval
2. FIT file parsing → device information extraction → model validation
3. Optional MQTT publishing with Home Assistant discovery
4. Logging and error handling throughout

### Webhook Processing
1. FastAPI server receives Strava webhook → activity created event
2. Fetch activity via Strava API → delegate to downstream sources
3. Parse FIT file → extract devices → publish via persistent MQTT client
4. Async processing to avoid blocking webhook responses

### NiceGUI Status UI
1. `ui.run_with(app)` binds NiceGUI routes to the existing FastAPI app (single server process)
2. `StatusViewModel` pulls data from `runtime_state.snapshot()` plus environment variables
3. `ui.timer(5.0, model.update)` keeps dashboard fields fresh
4. For interactive controls, add backend action callables in `register_status_page(...)` and invoke them from UI event handlers

### Local Development with Webhooks
Use localtunnel for public HTTPS endpoint during development:
```bash
# Start tunnel (helper script)
scripts/run-localtunnel.sh mystravawebhook

# Set webhook URL
export STRAVA_WEBHOOK_URL="https://mystravawebhook.loca.lt/strava/webhook"

# Start webhook server
uv run strava-sensor
```

This architecture enables flexible, extensible activity processing with robust error handling and Home Assistant integration.

- always run `uv run ./scripts/run-all-checks.sh` after you made changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CrashLoopBackCoffee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CrashLoopBackCoffee)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
