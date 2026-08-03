## aws-bench

> Operational guide for AI coding agents working in the **aws-bench** framework repository. For a human-facing overview, see [README.md](README.md).

# AGENTS.md

Operational guide for AI coding agents working in the **aws-bench** framework repository. For a human-facing overview, see [README.md](README.md).

aws-bench is a Python CLI and evaluation harness that runs AI coding agents against real, disposable AWS environments and scores the results. It is built on the [Harbor](https://github.com/harbor-framework/harbor) evaluation framework. Datasets (tasks + scenarios) live in the separate [aws-bench-datasets](https://github.com/aws-bench/aws-bench-datasets) repository.

## Setup

- **Python:** 3.12+
- **Package manager:** [uv](https://docs.astral.sh/uv/) (do not use `pip` directly)

```bash
uv sync                  # install dependencies (incl. dev group)
uv run aws-bench --help  # verify the CLI runs
```

## Commands

Run everything through `uv run` / `make` so the pinned toolchain is used.

```bash
make test         # uv run pytest --cov
make lint         # uv run ruff check .
make format       # uv run ruff format .
make format-check # uv run ruff format --check .
make typecheck    # uv run pyright
make check        # lint + format-check + typecheck + test  (no auto-fix; use in CI/pre-PR)
make fix          # ruff check --fix + ruff format          (auto-fix before committing)
make ready        # fix + check + registry-bump             (full local gate)
```

Run a single test:

```bash
uv run pytest tests/dataset/test_config_registry.py -k registry_mode -v
```

**Always run `make check` (or at least `make fix` then `make test`) before committing.** Fix all lint, type, and test failures.

## Project structure

```
aws_bench/
  cli/                 # Typer CLI entrypoints (aws-bench ...); main app in cli/main.py
  account_management/  # AWS Organizations / account provisioning
  resource_management/ # scenario deploy, verify, reset, cleanup, drift scanning
  scenario/            # scenario model + orchestration
  task/                # task model + execution
  dataset/             # dataset config + registry loading (config.py, registry.py)
  agents/              # agent adapters
  metrics/             # metrics collection
  utils/               # shared helpers (credentials, etc.)
  constants.py, exceptions.py, logging/
tests/                 # pytest suite (unit tests use moto to mock AWS)
scripts/               # update_registry.py and other tooling
registry.json          # GENERATED — see Boundaries
```

## Code style

- **Formatting & linting:** ruff (`make fix` before committing). Configuration lives in `pyproject.toml` under `[tool.ruff]`.
- **Type checking:** pyright must pass; the package ships `py.typed`. Add type hints to all new public functions.
- **Models:** use **Pydantic v2** for config and data models (validation + serialization).
- **CLI:** built with **Typer**; add new commands under `aws_bench/cli/` and wire them into `main.py`.
- **Async:** I/O against AWS and containers is async — follow the existing `async`/`await` patterns.
- **AWS SDK:** use `boto3`; prefer paginated calls and least-privilege describe/list operations.

Match the conventions of the surrounding code. aws-bench builds on Harbor — when extending Harbor interfaces, follow Harbor's conventions.

## Testing

- Framework: `pytest` (with `pytest-cov`, `pytest-asyncio`, `pytest-timeout`).
- AWS is mocked with **`moto`** in unit tests — **never call real AWS from unit tests.**
- Add or update tests for any behavior you change, and keep the suite green (`make test`).

## Git & PR workflow

- Use [Conventional Commits](https://www.conventionalcommits.org/) (e.g. `fix(cleanup): ...`, `docs(readme): ...`) — the repo uses `commitizen`.
- Keep PRs focused; run `make check` before opening one.
- See the org contribution guide: <https://github.com/aws-bench/aws-bench?tab=contributing-ov-file>.

## Boundaries

- ✅ **Always:** run `make fix` + `make check` before committing; add tests for changed behavior; use `uv`.
- ⚠️ **Ask first:** adding new dependencies; changing the CLI surface, public interfaces, or verifier/reward contract; modifying AWS provisioning, IAM, or teardown/cleanup logic.
- 🚫 **Never:**
  - Commit secrets, AWS credentials, or bearer tokens.
  - **Hand-edit `registry.json`** — it is generated. Run `make registry-bump` (`scripts/update_registry.py`) instead.
  - Run destructive operations against real AWS accounts you do not own, or outside a disposable aws-bench test environment.
  - Edit files under `.venv/` or other vendored/generated directories.

---
> Source: [aws-bench/aws-bench](https://github.com/aws-bench/aws-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
