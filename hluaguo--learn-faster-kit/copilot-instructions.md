## learn-faster-kit

> This is a Python 3.12 package using a `src/` layout. Runtime code lives in `src/learn_faster/`, with the CLI entry point in `src/learn_faster/cli/main.py`. Agent profiles are defined in `src/learn_faster/cli/agents.py`. Packaged learning assets live under `src/learn_faster/templates/`: shared scripts and references are in `templates/shared/`, while agent-specialized assets are in `templates/agents/<agent-name>/modes/<mode>/`. Keep user-facing documentation in `README.md`, release notes in `CHANGELOG.md`, and contributor process notes in `.github/CONTRIBUTING.md`.

# Repository Guidelines

## Project Structure & Module Organization

This is a Python 3.12 package using a `src/` layout. Runtime code lives in `src/learn_faster/`, with the CLI entry point in `src/learn_faster/cli/main.py`. Agent profiles are defined in `src/learn_faster/cli/agents.py`. Packaged learning assets live under `src/learn_faster/templates/`: shared scripts and references are in `templates/shared/`, while agent-specialized assets are in `templates/agents/<agent-name>/modes/<mode>/`. Keep user-facing documentation in `README.md`, release notes in `CHANGELOG.md`, and contributor process notes in `.github/CONTRIBUTING.md`.

There is currently no committed test suite. Add tests under `tests/` when changing behavior, mirroring the package structure where practical.

## Build, Test, and Development Commands

- `uv sync` installs project dependencies from `pyproject.toml` and `uv.lock`.
- `uv run learn-faster --help` runs the CLI locally through the project environment.
- `uv run learn-faster init --agent claude-code` initializes the current directory for the Claude Code profile.
- `uv run learn-faster init --agent codex` initializes the current directory for the Codex profile.
- `uv run learn-faster version` verifies the installed package metadata and console script.
- `uv build` builds source and wheel distributions using Hatchling.
- `uv run pytest` should be used once tests are added.

Avoid committing generated local install targets in consumer projects, such as `.learning/`, `.claude/`, `.codex/`, `CLAUDE.md`, or generated `AGENTS.md`. This repository's root `AGENTS.md` is the contributor guide and should stay tracked.

## Coding Style & Naming Conventions

Use standard Python style with 4-space indentation, type hints for new public helpers, and `pathlib.Path` for filesystem work. Prefer small, direct functions that match the current CLI style. Use `snake_case` for functions, variables, and module names; use `PascalCase` for classes. Agent names and user-facing command filenames should stay lowercase and hyphenated, for example `claude-code`, `codex`, and `generate-exam.md`.

Keep JSON output formatted with two-space indentation when matching existing generated files. Preserve Markdown frontmatter in template files when editing mode prompts or commands.

## Testing Guidelines

For CLI behavior, add focused tests that exercise argument handling, initialization decisions, and file generation. Mock subprocess calls to `claude` and avoid writing outside temporary directories. Name test files `test_<module>.py` and test functions `test_<behavior>()`. If a change updates templates, include assertions for copied filenames or rendered content where possible.

## Commit & Pull Request Guidelines

Use Conventional Commits, as documented in `.github/CONTRIBUTING.md`: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `build:`, `ci:`, or `chore:`. Recent history follows this pattern, for example `feat: add printable exam generator for exam mode`.

Pull requests should include a short description, motivation or linked issue, test results, and screenshots or terminal output when CLI behavior changes. Note any template changes explicitly because they affect files generated in downstream projects.

---
> Source: [hluaguo/learn-faster-kit](https://github.com/hluaguo/learn-faster-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
