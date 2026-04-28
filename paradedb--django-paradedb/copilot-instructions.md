## django-paradedb

> Run unit tests (no database required) before every push — this is what CI checks first:

# Testing

## Unit Tests

Run unit tests (no database required) before every push — this is what CI checks first:

```bash
uv run pytest -m "not integration"
```

## Integration Tests

Integration tests require a running ParadeDB/Postgres instance. Use the provided script to start the container and set up the correct DSN:

```bash
PARADEDB_PASSWORD=postgres scripts/run_integration_tests.sh
```

To run a subset of tests, pass pytest selectors:

```bash
PARADEDB_PASSWORD=postgres scripts/run_integration_tests.sh tests/integration/test_paradedb_queries.py::test_tokenizer_override_invalid_identifier
```

---
> Source: [paradedb/django-paradedb](https://github.com/paradedb/django-paradedb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
