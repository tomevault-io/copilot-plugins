## pamqp

> RabbitMQ-focused AMQP low-level library for encoding and decoding AMQP frames.

# pamqp

RabbitMQ-focused AMQP low-level library for encoding and decoding AMQP frames.

## Development

```bash
uv sync --all-groups        # Install dependencies
uv run coverage run         # Run tests with coverage
uv run coverage report      # View coverage report
uv run pre-commit run -a    # Run linting
```

## Code Style

- Ruff for linting and formatting (configured in pyproject.toml)
- Single quotes for strings
- 79 character line length
- `pamqp/commands.py` is auto-generated code — excluded from ruff formatting to preserve attribute ordering

## Architecture

- `pamqp/commands.py` — Auto-generated AMQP command classes (do not edit manually, use `tools/codegen.py`)
- `pamqp/encode.py` / `pamqp/decode.py` — Wire protocol encoding/decoding
- `pamqp/frame.py` — Frame marshaling/unmarshaling
- `pamqp/base.py` — Base classes for Frame and BasicProperties
- `pamqp/common.py` — Shared type aliases and struct helpers

---
> Source: [gmr/pamqp](https://github.com/gmr/pamqp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
