## coconut

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Coconut is a functional programming language that compiles to Python. It adds features like pattern matching, algebraic data types, partial application, pipeline operators, and lazy evaluation on top of Python syntax. The compiler performs one-pass compilation using PyParsing. If you are ever uncertain about what the intended behavior should be, refer to the documentation in DOCS.md.

## Common Commands

### Development Setup
```bash
make dev          # Install development environment with all dependencies and pre-commit hooks
```

### Testing

Tests can take a long time, especially when recompiling everything, so make sure to run them with a long timeout.

```bash
make test          # Recompile all test files and run them (should be used whenever testing any changes to the compiler itself)
make test-tests    # Just recompile changed test files and run them (much faster, but should only be used when the change being tested is just a change to the tests themselves)
```

### Running Coconut Code Directly
```bash
python -m coconut -c "your code here"    # Compile and execute Coconut code passed as a string
```

## Architecture

### Compilation Pipeline
1. **Pre-processing**: String replacement markers handle special cases (`coconut/compiler/compiler.py`)
2. **One-pass compilation**: PyParsing grammar with attached handlers transform Coconut to Python (`coconut/compiler/grammar.py`)
3. **Post-processing**: Generated header prepended to output (`coconut/compiler/header.py`)

### Core Modules

**`coconut/root.py`** - Python 2/3 compatibility layer. Every module imports `from coconut.root import *`. Defines version constants and base header.

**`coconut/constants.py`** - Global configuration: version info, platform detection flags (`PY26`...`PY314`, `WINDOWS`, `PYPY`, `CPYTHON`), dependencies, magic numbers, environment variables.

**`coconut/compiler/`**:
- `compiler.py` - `Compiler` class inheriting from `Grammar`. Main compilation logic.
- `grammar.py` - `Grammar` class with PyParsing rules and basic handlers.
- `matching.py` - `Matcher` class for pattern matching compilation (match statements, destructuring, pattern-matching functions).
- `header.py` - Generates runtime header from `templates/header.py_template`.

**`coconut/command/`**:
- `command.py` - `Command` class; CLI entry point via `start` method.
- `cli.py` - `ArgumentParser` for command-line arguments.
- `util.py` - `Prompt` (syntax-highlighted input), `Runner` (executes compiled Python).

**`coconut/icoconut/`** - Jupyter kernel implementation (subclasses IPython kernel).

### Key Entry Points
- `coconut/main.py` - Entry points for `coconut` and `coconut-run` CLI commands
- `coconut/api.py` - External API for programmatic usage
- `coconut/convenience.py` - High-level functions: `cmd()`, `version()`, `parse()`, `setup()`

## Testing

Tests are written in Coconut (`.coco` files), not Python. Located in `coconut/tests/src/`:
- `cocotest/agnostic/` - Version-agnostic tests (main.coco, suite.coco, util.coco)
- `cocotest/target_*/` - Version-specific tests

To add tests: write Coconut code with `assert` statements in the appropriate `.coco` test file.

## Code Conventions

- **Module imports**: Always include `from coconut.root import *` at the top for version compatibility
- **Logging**: Use `logger` from `coconut.terminal` for console output
- **Exceptions**: Custom hierarchy in `coconut/exceptions.py`
- **Author attribution**: Add yourself to "Authors:" in module docstrings when modifying files

## Branches
- `master` - Stable releases
- `develop` - Active development (default branch for contributions)

## Debugging GitHub Actions

### Viewing Logs for a Specific Job

When investigating a failed GitHub Actions job, use the GitHub API directly:

```bash
gh api repos/evhub/coconut/actions/jobs/{job_id}/logs
```

Replace `{job_id}` with the job ID from the URL (e.g., for `.../job/59132870840`, use `59132870840`).

**Note:** The more common `gh run view {run_id} --log-failed` command doesn't work if the overall run is still in progress, even if the specific job you want has already completed. Using the API directly bypasses this limitation.

---
> Source: [evhub/coconut](https://github.com/evhub/coconut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
