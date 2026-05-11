## sui-dev-skill

> This repository is documentation- and example-focused for Sui Move development.

# Repository Guidelines

## Project Structure & Module Organization
This repository is documentation- and example-focused for Sui Move development.
- `SKILL.md`: canonical skill definition and scope.
- `README.md`: quick-start overview and usage examples.
- `references/`: topic guides (`core_topics.md`, `object_model.md`, `ptb_guide.md`).
- `scripts/`: executable shell utilities and Move example modules (`*.sh`, `*.move`).

Place new runnable examples in `scripts/`. Place conceptual or long-form explanations in `references/`.

## Build, Test, and Development Commands
Use these commands from the repository root:
- `./scripts/setup_env.sh`: installs toolchain, validates setup, and optionally publishes.
- `sui move build`: compiles Move packages.
- `sui move test`: runs Move unit and scenario tests.
- `./scripts/deploy.sh 100000000`: builds and publishes with an explicit gas budget.
- `./scripts/version_check.sh old.mv new.mv`: compares compiled module bytecode.
- `./scripts/version_conflict_check.sh old_dir new_dir`: detects cross-version conflicts.

## Coding Style & Naming Conventions
- Shell scripts: `bash`, `#!/bin/bash`, and `set -Eeuo pipefail` for fail-fast behavior.
- Always quote shell variables (for example, `"$GAS_BUDGET"`), and return non-zero on errors.
- Move code: 4-space indentation, `snake_case` function names, `PascalCase` structs, lowercase module names (for example, `mypackage::advanced_examples`).
- Use `E...` `u64` constants for abort codes (for example, `ENotOwner`).
- Keep comments short and focused on object ownership, transfers, and invariants.

## Testing Guidelines
- Prefer in-module tests with `#[test]` and `sui::test_scenario`.
- Cover positive and negative flows; use `#[expected_failure(abort_code = ...)]` for expected aborts.
- Name tests `test_<behavior>` (for example, `test_reset_by_non_owner_fails`).
- Run `sui move test` before opening a PR. No fixed coverage threshold is defined, but all new logic should include tests.

## Commit & Pull Request Guidelines
- Follow the history style: concise, imperative subjects; prefixes like `docs:` and `fix:` are common.
- Keep commits focused (avoid mixing docs, scripts, and behavior changes when possible).
- PRs should include:
  - a clear summary of what changed and why
  - commands executed (`sui move build`, `sui move test`)
  - linked issue/PR context when applicable
  - relevant CLI output when script behavior changes

## Security & Configuration Tips
- Never commit private keys, wallet files, or local environment secrets.
- Verify `sui client active-address` and active network before publishing.
- Prefer explicit gas budgets instead of relying on defaults in deployment commands.

---
> Source: [Nebryx/sui-dev-skill](https://github.com/Nebryx/sui-dev-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
