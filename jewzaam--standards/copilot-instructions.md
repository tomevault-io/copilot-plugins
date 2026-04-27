## standards

> Reusable software development standards referenced by other projects via

# Standards Repository

Reusable software development standards referenced by other projects via
`~/source/standards/` in the user's global CLAUDE.md. This is a reference
library — nothing to build or deploy. Other projects read these files when
making decisions about style, structure, testing, and CI.

The full organized index with section descriptions is in
[README.md](README.md). Every file is also linked directly below for
tool-based lookup.

Run `make help` for available validation targets.

## Maintaining this file

Every non-infrastructure file in this repo must be linked directly from this
file. No file should require traversing intermediate documents to discover.

Run `make reachability` (or `python scripts/reachability.py --check`) to
verify. It fails if any content file is missing a direct link from CLAUDE.md
or README.md.

When adding a new standard or template:

1. Add the file
2. Add a direct link here under the appropriate section with a short description
3. Add a link in [README.md](README.md) under the matching section
4. Run `make reachability` to confirm

## Common

- [common/README.md](common/README.md)
- [common/naming.md](common/naming.md) — repo, package, module, variable naming conventions
- [common/versioning.md](common/versioning.md) — semver rules, version location in pyproject.toml and code
- [common/readme-format.md](common/readme-format.md) — README structure, badges, descriptions
- [common/submodules.md](common/submodules.md) — git submodule conventions and workflows
- [common/git-worktrees.md](common/git-worktrees.md) — worktree conventions, cleanup, AI-assisted parallel development
- [common/reachability.md](common/reachability.md) — document reachability enforcement from entry points

## Python

- [python/README.md](python/README.md)
- [python/style.md](python/style.md) — coding style, imports, type hints
- [python/project-structure.md](python/project-structure.md) — directory layout, required files
- [python/testing.md](python/testing.md) — pytest conventions, TDD, TEST_PLAN.md
- [python/complexity.md](python/complexity.md) — cyclomatic complexity limit (10), ruff C901 enforcement
- [python/subprocess-security.md](python/subprocess-security.md) — subprocess and localhost server security rules
- [python/cross-platform.md](python/cross-platform.md) — making python/python3 work across Linux, macOS, Windows
- [python/shared-venv.md](python/shared-venv.md) — shared `~/.venv/ap/` for ap-* projects, local `.venv` for others
- [python/ap-common-usage.md](python/ap-common-usage.md) — shared constants from ap-common
- [python/logging-progress.md](python/logging-progress.md) — logger setup, `--log-file`, progress bars
- [python/settings-persistence.md](python/settings-persistence.md) — dataclass settings with atomic JSON I/O
- [python/agent-sdk.md](python/agent-sdk.md) — Claude Agent SDK integration, threading with tkinter, permission control

## Tkinter UI

- [python/tkinter/README.md](python/tkinter/README.md)
- [python/tkinter/architecture.md](python/tkinter/architecture.md) — app structure, controller pattern, window hierarchy
- [python/tkinter/windows.md](python/tkinter/windows.md) — window lifecycle, borderless mode, dragging, position persistence
- [python/tkinter/widgets.md](python/tkinter/widgets.md) — layout, StringVar, styling, fonts, context menus
- [python/tkinter/threading.md](python/tkinter/threading.md) — thread-safe UI updates, daemon threads, periodic polling
- [python/tkinter/dialogs.md](python/tkinter/dialogs.md) — modal dialogs, settings windows, color pickers
- [python/tkinter/testing.md](python/tkinter/testing.md) — two-layer testing strategy, root fixture, headless CI

## Python Templates

- [python/templates/Makefile](python/templates/Makefile) — standard Makefile with all required targets
- [python/templates/pyproject.toml](python/templates/pyproject.toml) — package config with standard dev dependencies
- [python/templates/TEST_PLAN.md](python/templates/TEST_PLAN.md) — testing strategy documentation template
- [python/templates/pipx.mk](python/templates/pipx.mk) — CLI tool installation via pipx (includable `.mk`)
- [python/templates/version-check.mk](python/templates/version-check.mk) — semver validation (includable `.mk`)
- [python/templates/version-check.sh](python/templates/version-check.sh) — shell script for semver validation

## CLI

- [cli/README.md](cli/README.md)
- [cli/conventions.md](cli/conventions.md) — argument naming, flags, `--log-file`, `--debug`
- [cli/testing.md](cli/testing.md) — testing main() with sys.argv patching

## Build and CI/CD

- [build/README.md](build/README.md)
- [build/makefile.md](build/makefile.md) — Makefile conventions, required targets, `PACKAGE_NAME`, `VENV_DIR`
- [build/github-workflows.md](build/github-workflows.md) — workflow conventions, Python versions, triggers
- [build/local-workflow-testing.md](build/local-workflow-testing.md) — testing GitHub Actions workflows locally with act, safe defaults
- [build/templates/workflows/test.yml](build/templates/workflows/test.yml) — pytest on Python 3.12-3.14
- [build/templates/workflows/lint.yml](build/templates/workflows/lint.yml) — flake8 linter
- [build/templates/workflows/typecheck.yml](build/templates/workflows/typecheck.yml) — mypy type checker
- [build/templates/workflows/format.yml](build/templates/workflows/format.yml) — black formatting check
- [build/templates/workflows/coverage.yml](build/templates/workflows/coverage.yml) — 80% coverage threshold
- [build/templates/workflows/mutation.yml](build/templates/workflows/mutation.yml) — mutmut mutation testing
- [build/templates/workflows/reachability.yml](build/templates/workflows/reachability.yml) — document reachability check
- [build/templates/workflows/version-check.yml](build/templates/workflows/version-check.yml) — semver validation (optional)

## Claude Code

- [claude-code/skills.md](claude-code/skills.md) — authoring Claude Code skills (SKILL.md files)
- [claude-code/hook-state-transitions.md](claude-code/hook-state-transitions.md) — hook event types, state machines, configuration

## Planned (empty)

- [android/README.md](android/README.md) — Android development standards (placeholder)
- [mobile/README.md](mobile/README.md) — cross-platform mobile patterns (placeholder)
- [guides/README.md](guides/README.md) — process and tooling how-tos (placeholder)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jewzaam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
