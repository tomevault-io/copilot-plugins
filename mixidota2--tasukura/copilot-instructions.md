## tasukura

> Follow [CONTRIBUTING.md](./CONTRIBUTING.md). In particular:

# Development Workflow

Follow [CONTRIBUTING.md](./CONTRIBUTING.md). In particular:

- Never commit directly to main. Always create a feature branch first.
- Run all CI checks before pushing and ensure they pass:
  - `uv run pytest -v`
  - `uv run ruff check src/ tests/`
  - `uv run ruff format --check src/ tests/`
  - `uv run ty check src/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mixidota2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
