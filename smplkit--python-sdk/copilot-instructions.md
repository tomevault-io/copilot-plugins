## python-sdk

> See `~/.claude/CLAUDE.md` for universal rules (git workflow, testing, code quality, SDK conventions, etc.).

# smplkit Python SDK

See `~/.claude/CLAUDE.md` for universal rules (git workflow, testing, code quality, SDK conventions, etc.).

## Repository Structure

Two-layer architecture per ADR-021:
- `src/smplkit/_generated/` — Auto-generated client code from OpenAPI specs. Do not edit manually.
- `src/smplkit/` (excluding `_generated/`) — Hand-crafted SDK wrapper. This is the public API.

## Regenerating Clients

```bash
make generate
```

Spec updates are automated: each source repo (app, config, flags, logging) pushes spec changes to this repo via PR. PRs from `regen/` branches authored by `notmikegorman` are auto-merged after CI passes (see `.github/workflows/auto-merge.yml`).

## Testing

```bash
pytest --cov=smplkit --cov-report=term-missing
```

## Python Version Policy

The SDK supports Python 3.10 through 3.13. Development uses Python 3.13 (the latest stable).

- `requires-python = ">=3.10"` in pyproject.toml is the enforced minimum.
- CI runs the full test suite against 3.10, 3.11, 3.12, and 3.13 on every push.
- Do NOT use Python features introduced after 3.10 unless guarded by a version check or `typing_extensions` backport. Specifically avoid:
  - `ExceptionGroup` / `except*` (3.11+)
  - `type` statement for type aliases (3.12+)
  - `typing.override` without `typing_extensions` fallback (3.12+)
  - `itertools.batched` (3.12+)
  - `pathlib.Path.walk` (3.12+)
- PEP 604 union syntax (`X | Y`) IS safe — available from 3.10.
- `from __future__ import annotations` IS safe for forward references.
- When adding new dependencies, verify their minimum Python version is <= 3.10.

## Package Naming

- **PyPI project name:** `smplkit-sdk` (install via `pip install smplkit-sdk`)
- **Python package name:** `smplkit` (import via `from smplkit import SmplkitClient`)
- The PyPI name `smplkit` is taken by an unrelated project.

## Publishing

- Version is derived from git tags via `hatch-vcs`. Do not set version in pyproject.toml.
- Publishes to PyPI via OIDC trusted publishing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smplkit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/smplkit)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
