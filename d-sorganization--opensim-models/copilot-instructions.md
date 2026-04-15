## opensim-models

> Python library for generating OpenSim (.osim) musculoskeletal models of classical

# CLAUDE.md -- OpenSim_Models

## What This Is

Python library for generating OpenSim (.osim) musculoskeletal models of classical
barbell exercises and human movement patterns (squat, deadlift, bench press, snatch,
clean & jerk, gait, sit-to-stand). Models are built programmatically using
XML-generation utilities and validated against OpenSim's schema.

## Key Directories

- `src/opensim_models/` -- main package (exercises, optimization, shared utilities, visualization)
- `tests/` -- pytest test suite (unit, integration, parity, hypothesis)
- `examples/` -- example scripts for generating models
- `rust_core/` -- Rust acceleration core (Cargo project, future use)
- `scripts/` -- dev setup scripts
- `.github/workflows/` -- CI configuration

## Development Commands

```bash
ruff check src scripts tests              # lint
ruff format --check src scripts tests      # format check
ruff format src scripts tests              # auto-format
mypy src --config-file pyproject.toml      # type check
python -m pytest -n auto --timeout=60 \
  -m "not slow and not requires_opensim" \
  --cov=src --cov-fail-under=80 -v         # run tests with coverage
```

## CI Requirements

1. `ruff check` -- zero violations
2. `ruff format --check` -- zero diffs
3. `mypy src` -- zero errors
4. `pytest` with **80% coverage minimum**
5. No `TODO`/`FIXME` in source (except scripts/)
6. `pip-audit` security scan
7. `bandit` security scan on src/

## Coding Standards

- **Formatter/Linter:** Ruff (line-length 88)
- **Type checking:** mypy with `check_untyped_defs = true`
- **Logging:** Use `logging` module, never `print()` in `src/`
- **Logging format:** Use `%s` style, not f-strings in log calls
- **DbC:** Preconditions/postconditions in `shared/contracts/`
- **DRY:** Extract shared logic into `shared/` utilities
- **Testing:** pytest, hypothesis for property-based tests, 80% coverage floor
- **No wildcard imports**, catch specific exceptions, use type hints

## Architecture

- Each exercise lives in `src/opensim_models/exercises/<name>/`
- Shared body model, barbell model, XML helpers in `src/opensim_models/shared/`
- Optimization objectives in `src/opensim_models/optimization/`
- Visualization (matplotlib) in `src/opensim_models/visualization/`
- Exercise registry: `EXERCISE_BUILDERS` dict in `exercises/__init__.py`

## Git Workflow

- Conventional Commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`
- Branch naming: `feat/description`, `fix/description`
- PRs must pass CI before merge

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/D-sorganization)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/D-sorganization)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
