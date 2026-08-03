## csas-ex-xray-extractor

> > AI agent instructions for this repository. See also: [src/component.py](src/component.py), [src/xray_api.py](src/xray_api.py), [src/configuration.py](src/configuration.py).

# AGENTS.md — Xray Extractor

> AI agent instructions for this repository. See also: [src/component.py](src/component.py), [src/xray_api.py](src/xray_api.py), [src/configuration.py](src/configuration.py).

## Project Overview

Keboola extractor: reads CSV, queries Xray API for test counts per row, writes output CSV with results.

## Build & Run

```bash
# Build and run locally (Docker)
docker-compose build
docker-compose run --rm dev

# Lint + unit tests (Docker)
docker-compose run --rm test

# Without Docker
flake8 --config=flake8.cfg
python -m unittest discover
```

Entrypoint: `python -u /code/src/component.py`. Requires `KBC_DATADIR` env var (defaults to `./data`).

## Architecture

| File | Role |
|------|------|
| [`src/component.py`](src/component.py) | `Component(ComponentBase)` — pipeline: config → auth → read CSV → call API per row → write output |
| [`src/configuration.py`](src/configuration.py) | Pydantic config model. Encrypted Keboola params use `#` prefix aliases (e.g. `#xray_client_id`) |
| [`src/xray_api.py`](src/xray_api.py) | `XrayApiClient` — bearer token auth + GraphQL queries, retry on 429/5xx (4 attempts, 15s linear backoff) |
| [`component_config/configSchema.json`](component_config/configSchema.json) | JSON Schema for the Keboola UI form |
| [`data/`](data/) | Local test data (`config.json`, `in/`, `out/`) — used with `KBC_DATADIR=./data` |

## Python Conventions

See [.github/instructions/python-conventions.instructions.md](.github/instructions/python-conventions.instructions.md).

## Business Logic

- **Row filtering**: Rows are processed only when both `AUTE_DATA_AUTOMATICALLY = "Y"` **and** `IS_VALID = "Y"`. These are the real column names — do not rename or "fix" them.
- **Input format**: Each source column contains a JSON array of exactly 3 elements: `[project_id, folder_path, jql_query]`.
- **Two column pairs**: The component processes total tests and automated tests independently per row (four column names configured in `configSchema.json`).
- **Comments**: Not verbose, Concise, in English.
- **No over-engineering**: Only implement what is asked. No extra abstractions, docstrings, or error handling for impossible scenarios.

## Development Guidelines

- Be critical and direct. Identify real trade-offs; do not just validate proposals.
- If a better approach exists or a hidden problem is present, say so clearly.
- If complexity exceeds benefit, recommend against and explain why.
- Before each action, give a brief comment on what you are doing and why.

---
> Source: [mihavlik-cen58506/csas.ex-xray-extractor](https://github.com/mihavlik-cen58506/csas.ex-xray-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
