## pip-select

> This repository is a small, single-file Python CLI for interactively upgrading

# Agent Instructions for pip-select

This repository is a small, single-file Python CLI for interactively upgrading
pip-installed packages while excluding conda-installed ones.

The instructions below should match the repository as it exists today, not an
idealized future state.

## Current Repository Layout

- `pip-select.py` - the entire application
- `README.md` - user-facing documentation
- `AGENTS.md` - contributor/agent guidance

There is currently no committed test suite, no `pyproject.toml`, and no
packaging/module layout beyond the standalone script. Generated artifacts such
as `__pycache__/` or `.DS_Store` are not part of the source tree and should not
be treated as code.

## What the Script Does Today

The current script:

- detects whether it is running inside a conda environment
- excludes conda-installed packages using two signals:
  - `INSTALLER` metadata
  - `conda-meta/*.json`
- calls `python -m pip list --outdated --format=json`
- parses outdated package data into `UpgradeCandidate` records
- presents either:
  - a curses multi-select UI, or
  - a text fallback selector
- upgrades the selected packages with `python -m pip install --upgrade`

## Current CLI Contract

Preserve these behaviors unless the user explicitly asks to change them:

- Supported flags:
  - `--user`
  - `--dry-run`
  - `--no-curses`
- Extra pip arguments are passed through after `--`
- The script uses `sys.executable -m pip`, not a hardcoded `pip` binary
- `--user` inside a virtual environment returns exit code `2`
- Cancelling selection returns exit code `2`
- No upgradeable packages returns exit code `0`
- Unknown/non-conda installers are currently treated as pip-like by default
- The progress bar is time-based animation, not true pip progress reporting

## Safe Validation Commands

Use commands that work with the repository's current structure.

### Always Safe

```bash
python3 -m py_compile pip-select.py
python3 pip-select.py --help
```

### Safe Behavior Smoke Test

```bash
python3 pip-select.py --dry-run --no-curses
```

Notes:

- `--dry-run` avoids installs, but the command still inspects the environment
  and queries pip for outdated packages
- this may hit the network, depending on local pip behavior and package index
  access

### Optional Local Tooling

These commands are useful if the tools are installed locally, but the
repository does not currently commit configuration for them:

```bash
ruff check pip-select.py
black --check pip-select.py
mypy pip-select.py
```

Do not assume these tools are available in every environment.

## Code Map

### Helpers and Normalization

- `norm_name()`
- `run_capture()`
- `run_stream()`
- `ask_yes_no()`
- `is_tty()`
- `in_venv()`

### Conda Detection and Package Classification

- `detect_conda_prefix()`
- `conda_meta_names()`
- `read_installer()`
- `list_installed_distributions()`
- `pip_installed_set_excluding_conda()`

### Upgrade Discovery

- `parse_pip_list_outdated_json()`
- `_show_progress_bar()`
- `get_upgrade_candidates_from_pip()`

### Selection UI

- `curses_select()`
- `fallback_select()`

### Upgrade Execution and CLI Entry Point

- `upgrade_selected()`
- `main()`

## Editing Guidelines

### Preserve the Project Shape

- Keep the tool dependency-free
- Keep the single-file application structure unless explicitly asked to split it
- Maintain Python 3.8+ compatibility
- Maintain Linux/macOS assumptions for curses behavior

### Match Existing Code Style

Follow the patterns already used in `pip-select.py`:

- `from __future__ import annotations`
- standard-library-only imports
- type hints using `Optional`, `List`, `Dict`, `Set`, `Tuple`, `Sequence`
- `@dataclass(frozen=True)` for simple records
- double-quoted strings
- section comments in this form:

```python
# ----------------------------
# Section Name
# ----------------------------
```

Do not introduce unrelated style churn while editing a small feature or fix.

### Subprocess and Environment Rules

- Prefer `subprocess.run()` wrappers already present in the file
- Reuse `_base_env()` for pip-related subprocesses
- Preserve `sys.executable -m pip`
- Surface subprocess failures clearly; do not silently swallow them

### UI and UX Rules

- Keep both the curses UI and text fallback working
- Preserve TTY-aware behavior
- Treat blank fallback input as cancel
- Avoid adding dependencies for richer UI

## Testing and Change Strategy

Because there is no committed automated test suite right now:

- make focused, surgical changes
- validate syntax after code edits
- run `--help` after CLI changes
- run `--dry-run --no-curses` for behavior changes when safe
- manually reason through exit codes and selection flow when touching control
  flow

If you add tests in the future, do not assume a normal import path from the
script name alone; `pip-select.py` contains a hyphen, so direct module imports
need care. Subprocess-based tests or explicit file-based import helpers are
safer than assuming `import pip-select` works.

## Documentation Expectations

Update `README.md` when user-visible behavior changes, especially for:

- command-line flags
- selection behavior
- conda filtering behavior
- examples and troubleshooting text

Update `AGENTS.md` when the repository structure or recommended validation flow
changes.

## Practical Reminders for Agents

- Favor backward-compatible CLI changes
- Do not add external runtime dependencies
- Do not convert this into a package/module layout unless asked
- Be careful with behavior changes around conda detection and installer
  classification
- Prefer small helpers over large refactors
- Clean up temporary files you create during your work

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rsnemmen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
