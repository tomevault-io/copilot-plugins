## cts-gym

> - `legged_gym/`: Core library code.

# Repository Guidelines

## Project Structure & Module Organization
- `legged_gym/`: Core library code.
- `legged_gym/envs/`: Base environment and Unitree Go2 task configuration.
- `legged_gym/algorithm/`: PPO and model components.
- `legged_gym/utils/`: Helpers (task registry, terrain, logging).
- `legged_gym/scripts/`: Entrypoints for training and evaluation.
- `legged_gym/tests/`: Lightweight environment test(s).
- `resources/`: Robot assets, meshes, URDFs.
- `logs/`: Training outputs and checkpoints.

## Build, Test, and Development Commands
- `pip install -e .`: Install the package in editable mode.
- `python -m legged_gym.scripts.train --task=go2`: Train a policy.
- `python -m legged_gym.scripts.train --task=go2 --resume`: Resume training.
- `python -m legged_gym.scripts.play --task=go2`: Visualize a trained policy.
- `python -m legged_gym.tests.test_env --task=go2`: Run the environment smoke test.

## Coding Style & Naming Conventions
- Python code uses 4-space indentation and module-based organization.
- Follow existing patterns: `snake_case` for functions/vars, `CamelCase` for classes.
- No formatter or linter is enforced in-repo; keep changes consistent with nearby files.

## Testing Guidelines
- Tests are in `legged_gym/tests/`, with entrypoints like `test_env.py`.
- Prefer targeted, fast tests that validate environment setup and key invariants.
- Name new tests `test_*.py` and keep them runnable via `python -m legged_gym.tests.<name>`.

## Commit & Pull Request Guidelines
- Commit messages follow a conventional style (e.g., `feat:`, `fix:`, `chore:`).
- PRs should include a clear summary, testing notes, and linked issues when applicable.
- Include artifacts when relevant (e.g., training curves or logs under `logs/`).

## Notes for Contributors
- Task registry entries live in `legged_gym/utils/task_registry.py` and should be updated when adding new tasks or configs.
- Checkpoints are saved under `logs/<experiment>/<timestamp>/model_*.pt`.

---
> Source: [JerryGong0911/cts_gym](https://github.com/JerryGong0911/cts_gym) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
