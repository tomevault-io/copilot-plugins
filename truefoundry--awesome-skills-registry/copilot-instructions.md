## awesome-skills-registry

> This repository maintains an agent-agnostic skills registry. Core files live at the repo root: `sources.yaml` lists GitHub repositories to scan, `pyproject.toml` and `uv.lock` define the Python 3.12 environment, and `README.md` documents registry behavior. Generated skill metadata lives under `registry/{owner}-{repo}/{owner}-{repo}-{skill-dir}.yaml` (one folder per source repo). Automation lives in `scripts/`: `update.py` scans configured sources and writes registry entries, while `lint_registry.py` validates and optionally fixes registry YAML. GitHub Actions are in `.github/workflows/`; the local pre-commit hook is `.githooks/pre-commit`.

# Repository Guidelines

## Project Structure & Module Organization

This repository maintains an agent-agnostic skills registry. Core files live at the repo root: `sources.yaml` lists GitHub repositories to scan, `pyproject.toml` and `uv.lock` define the Python 3.12 environment, and `README.md` documents registry behavior. Generated skill metadata lives under `registry/{owner}-{repo}/{owner}-{repo}-{skill-dir}.yaml` (one folder per source repo). Automation lives in `scripts/`: `update.py` scans configured sources and writes registry entries, while `lint_registry.py` validates and optionally fixes registry YAML. GitHub Actions are in `.github/workflows/`; the local pre-commit hook is `.githooks/pre-commit`.

## Build, Test, and Development Commands

- `uv sync --locked`: install the locked Python dependencies.
- `GITHUB_TOKEN=<token> uv run python scripts/update.py`: scan `sources.yaml`, add missing skills, and create a registry update PR.
- `uv run python scripts/lint_registry.py`: validate all registry YAML files.
- `uv run python scripts/lint_registry.py --fix`: normalize fixable YAML issues, especially description whitespace.
- `git config core.hooksPath .githooks`: enable the local staged-registry lint hook.

## Coding Style & Naming Conventions

Use Python 3.12 with type hints and simple dataclasses where appropriate. Keep scripts small, procedural, and explicit; prefer standard-library utilities and existing dependencies (`PyGithub`, `PyYAML`) over new custom machinery. Registry filenames must follow `{owner}-{repo}-{skill_dir}.yaml`; IDs use the same stem. Each skill lives in a per-source folder named `{owner}-{repo}`.

## Testing Guidelines

There is no dedicated test suite yet. Treat `uv run python scripts/lint_registry.py` as the required validation for registry changes. For script behavior changes, add focused `pytest` tests before broad refactors; keep fixtures small and representative of `sources.yaml` and registry YAML entries.

## Commit & Pull Request Guidelines

History uses conventional commits such as `chore: use uv for python deps`, `feat(registry): add skill scanner and weekly pipeline`, and `chore(registry): add ... skill entries`. Keep commit messages brief and scoped. PRs should explain whether they modify sources, generated registry entries, or automation. Include the validation command run, link related issues when available, and call out any generated files or manual edits to registry YAML.

## Security & Configuration Tips

Never commit tokens. `scripts/update.py` requires `GITHUB_TOKEN`; pass it through the environment. Use least-privilege GitHub tokens where possible: `public_repo` for public-only scans, broader `repo` access only when private repositories are needed.

---
> Source: [truefoundry/awesome-skills-registry](https://github.com/truefoundry/awesome-skills-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
