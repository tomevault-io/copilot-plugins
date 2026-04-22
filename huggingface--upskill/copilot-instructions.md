## upskill

> - Target Python `3.13`.

# Agent guidance

## Code quality baseline

- Target Python `3.13`.
- Keep changes compatible with the repo's `pyproject.toml` settings.
- Prefer small, typed, composable functions over large command-style blocks.
- Avoid introducing new complexity suppressions unless there is a strong reason.

## Required local checks

Before finishing a change, run:

```bash
uv sync --extra dev
uv run scripts/format.py
uv run scripts/lint.py
uv run scripts/typecheck.py
uv run --extra dev pytest -v
```

CI enforces the same flow in `.github/workflows/ci.yml`.

## Ruff rules

Formatting and linting are enforced with Ruff.

- Line length: `100`
- Target version: `py313`
- Enabled lint families:
  - `B` - bugbear
  - `C90` - cyclomatic complexity
  - `E` - pycodestyle errors
  - `F` - pyflakes
  - `I` - import sorting
  - `RUF` - Ruff-specific rules
  - `SIM` - simplifications
  - `TCH` - type-checking import hygiene
  - `UP` - pyupgrade
- `E501` is ignored; let `ruff format` own line wrapping.
- Cyclomatic complexity limit: `15`

## Type checking

- `ty` is required for `src`, `tests`, and `scripts`.
- Add or improve annotations when touching code that is ambiguous to the type checker.
- Prefer explicit protocols / typed helper structures over `object` when wiring dynamic APIs.
- Keep type-only imports behind `TYPE_CHECKING` when appropriate.

## Tests

- Add or update tests for behavior changes.
- Keep the test suite passing with `pytest`.
- Use focused unit-style tests for logic changes when possible.

## Practical authoring guidance

- Prefer refactoring over adding broad ignores.
- If a function is nearing the complexity limit, split it before adding more branches.
- Keep CLI orchestration, model resolution, and persistence logic separated when possible.
- When adding developer tooling, update the README and CI together.

---
> Source: [huggingface/upskill](https://github.com/huggingface/upskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
