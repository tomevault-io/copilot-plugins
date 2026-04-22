## alchemist

> Linting rules for src/trading_server/ Trading server. Matches CI workflow configuration exactly.


# Python Trading Server Linting Rules (`src/trading_server/`)

## Overview

The Trading server is linted in CI with: **Black**, **Flake8**, and **MyPy**.

**Note**: Bandit and Safety are installed in CI but not actually run. Only the three tools above are executed.

## CI Configuration

See `.github/workflows/lint_server.yml` for the exact CI configuration.

## Linting Tools

### 1. Black (Code Formatting)
- **Line length**: 120 characters (default Black is 88, but project uses 120)
- **CI command**: `black --check src/trading_server/src` (from project root)
- **Local command**: `python -m black --check src/` (from `src/trading_server/` directory)
- **Auto-fix**: `python -m black src/` (from `src/trading_server/` directory)

### 2. Flake8 (Style Checking)
- **Max line length**: 120 characters
- **Ignore**: E203 (whitespace before ':'), W503 (line break before binary operator)
- **CI command**: `flake8 src/trading_server/src --max-line-length=120 --extend-ignore=E203,W503` (from project root)
- **Local command**: `python -m flake8 src/ --max-line-length=120 --extend-ignore=E203,W503` (from `src/trading_server/` directory)
- **Auto-fix**: Use `autopep8` or fix manually

### 3. MyPy (Type Checking)
- **Ignore missing imports**: Yes (for third-party libraries)
- **CI command**: `mypy src/trading_server/src --ignore-missing-imports` (from project root)
- **Local command**: `python -m mypy src/ --ignore-missing-imports` (from `src/trading_server/` directory)

## Quick Commands

### Run All Checks (Recommended)
```bash
cd src/trading_server
./lint.sh
```

This script runs exactly the same checks as CI:
1. Black formatting check
2. Flake8 style check
3. MyPy type check

### Manual Commands
```bash
cd src/trading_server

# Format code
python -m black --check src/

# Check style
python -m flake8 src/ --max-line-length=120 --extend-ignore=E203,W503

# Type check
python -m mypy src/ --ignore-missing-imports
```

### Auto-Fix
```bash
cd src/trading_server
python -m black src/  # Auto-format code
python -m isort src/  # Organize imports (optional but recommended)
# Then manually fix remaining flake8 and mypy errors
```

## Project-Specific Rules

### Trading Code Requirements
- ✅ Trading-related code must have proper error handling
- ✅ All financial calculations must be type-checked
- ✅ MT5 connector code should handle connection failures gracefully
- ✅ Risk management code must be thoroughly tested

### Code Quality Standards
- ✅ No unused imports
- ✅ No unused variables (prefix with `_` if intentionally unused)
- ✅ No bare `except:` clauses - use specific exception types
- ✅ Add docstrings to all public functions and classes
- ✅ Import organization: stdlib → third-party → local

## Before Committing

Ensure all checks pass:
```bash
cd src/trading_server
./lint.sh
```

The script will exit with an error if any check fails. Fix errors and re-run until all checks pass.

## CI Workflow Details

The CI workflow (`.github/workflows/lint_server.yml`) runs these exact commands:
1. Installs dependencies: `pip install black flake8 mypy` (Bandit and Safety are installed but not used)
2. Runs: `black --check src/trading_server/src` (from project root)
3. Runs: `flake8 src/trading_server/src --max-line-length=120 --extend-ignore=E203,W503` (from project root)
4. Runs: `mypy src/trading_server/src --ignore-missing-imports` (from project root)

Note: The CI runs commands from the project root, but the local `lint.sh` script runs from `src/trading_server/` directory, so paths are adjusted accordingly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxnorm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
