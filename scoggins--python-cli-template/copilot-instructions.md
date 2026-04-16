## python-cli-template

> This file provides guidance for AI assistants working in this repository.

# CLAUDE.md

This file provides guidance for AI assistants working in this repository.

## Repository Overview

This is a **minimal Python CLI template** — a skeleton project intended as a starting point for building Python command-line tools. It provides a standard CLI structure with argument parsing, logging, and config file loading that developers fork and extend.

## Repository Structure

```
python-cli-template/
├── python-cli.py      # Main entry point — the CLI application skeleton
├── .gitignore         # Standard Python gitignore
└── CLAUDE.md          # This file
```

## Main File: `python-cli.py`

The sole source file. It demonstrates the intended patterns for any CLI built from this template:

- **Argument parsing** via `argparse` with three pre-wired flags:
  - `-c / --config <file>`: Path to an INI config file (default: `config.ini`)
  - `--debug`: Enables `DEBUG`-level logging (default is `INFO`)
  - `--commit`: "Dry-run" safety flag — when omitted, log lines are prefixed with `[NO COMMIT]` to make it clear no changes are being made
- **Logging** configured via `logging.basicConfig` using a `{`-style format string
- **Config loading** via `configparser.ConfigParser.read_file()` — expects a standard INI file
- **Custom code placeholder** at line 46 (`# Custom Code Here`) — this is where application logic goes

### Running the Script

```bash
# Dry run with default config.ini
python python-cli.py

# With a specific config file
python python-cli.py -c myconfig.ini

# Enable debug output
python python-cli.py --debug

# Actually apply changes (disables [NO COMMIT] prefix)
python python-cli.py --commit

# Combine flags
python python-cli.py -c myconfig.ini --debug --commit
```

A `config.ini` file must exist (or be specified via `-c`) — the script will fail if the config file cannot be opened, because `argparse.FileType('r')` opens the file at parse time.

## Key Conventions

### The `--commit` / dry-run pattern

This template bakes in a dry-run safety pattern. Any script derived from this template should:

1. Default to **not** making changes (`--commit` defaults to `False`)
2. Gate all write/mutating operations behind `if args.commit:`
3. Log what *would* happen even in dry-run mode, so the `[NO COMMIT]` prefix in log output makes the intent clear

### Logging style

Use the module-level `logging` functions (`logging.info()`, `logging.debug()`, etc.) throughout — do not instantiate named loggers unless the project grows to multiple modules. The format uses `{`-style (not `%`-style) placeholders.

### Config file

The config file is expected to be INI format, read via `configparser`. Add sections and keys to `config.ini` (not tracked in this repo) and access them as `config['section']['key']`.

## Development Setup

This template has **no package management or dependency files** yet. When extending it:

1. Create a virtual environment:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```
2. Add dependencies to a `requirements.txt` (for scripts) or `pyproject.toml` (for packages)
3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Testing

No test suite is configured. When adding tests:

- Use `pytest` (the gitignore already accounts for `.pytest_cache/`)
- Place tests in a `tests/` directory
- Run with `pytest`

## Linting and Formatting

No linters are configured. Recommended additions for new projects based on this template:

- **ruff** for linting and import sorting
- **black** or **ruff format** for formatting
- **mypy** for type checking (`.mypy_cache/` is already gitignored)

## What to Do When Extending This Template

1. Rename `python-cli.py` to reflect the actual tool name
2. Update the `description=` string in `ArgumentParser`
3. Add custom arguments as needed
4. Replace the `# Custom Code Here` comment with actual logic
5. Create a `config.ini` (and a `config.ini.example` to commit)
6. Add `requirements.txt` or `pyproject.toml` with any dependencies
7. Set up tests, linting, and CI as appropriate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scoggins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
