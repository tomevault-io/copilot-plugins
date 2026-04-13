## pyshell

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PyShell is a Python library that provides a simple interface for executing shell commands as Python functions with proper error handling. The library focuses on simplicity and explicit error handling, making it different from the existing `pyshell` package on PyPI.

## Architecture

The codebase follows a minimal architecture:

```
src/pyshell/
├── __init__.py    # Exports shell() function and ShellError exception
└── core.py        # Core implementation with subprocess wrapper
```

**Key Components:**

1. **`shell()` function** (src/pyshell/core.py:10-55): Main entry point that executes shell scripts
   - Wraps subprocess.run with `set -e` for error propagation
   - Captures output and raises ShellError on non-zero exit codes
   - Supports environment variable injection and subprocess.run kwargs

2. **`ShellError` exception** (src/pyshell/core.py:3-8): Custom exception for failed commands
   - Provides returncode, stderr, and stdout attributes
   - Includes informative error messages with script and stderr previews

## Development Commands

Since this is a minimal Python package without test infrastructure:

```bash
# Install the package in development mode
pip install -e .

# Build the package
pip install build
python -m build

# Run example scripts
python examples/success.py
python examples/fail.py
```

## Important Implementation Details

- The library uses `set -e` in shell scripts to ensure error propagation
- All shell commands are executed with `shell=True` in subprocess
- Environment variables can be merged with existing ones via the `env` parameter
- The library strips output by default and provides text mode output
- Error messages include script preview (first 100 chars) and stderr preview (first 200 chars)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/crimson206)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/crimson206)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
