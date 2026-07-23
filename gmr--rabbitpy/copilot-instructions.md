## rabbitpy

> A pure python, thread-safe, minimalistic and pythonic RabbitMQ client library.

# rabbitpy

A pure python, thread-safe, minimalistic and pythonic RabbitMQ client library.

## Development

```bash
uv sync --all-groups        # Install all dependencies (dev + docs)
uv run coverage run         # Run tests with coverage
uv run coverage report      # View coverage report
uv run pre-commit run -a    # Run linting
uv run mkdocs serve         # Serve docs locally at http://127.0.0.1:8000
```

## Integration Tests

Integration tests require a running RabbitMQ instance. Use Docker Compose to start one:

```bash
docker compose up -d
./bootstrap.sh
```

This creates `build/test.env` with the RabbitMQ connection URLs needed by the integration tests.

## Code Style

- Ruff for linting and formatting (configured in pyproject.toml)
- Single quotes for strings
- 79 character line length

---
> Source: [gmr/rabbitpy](https://github.com/gmr/rabbitpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
