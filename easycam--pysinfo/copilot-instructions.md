## pysinfo

> This document provides guidelines for agentic coding agents working on the PySInfo codebase.

# AGENTS.md - PySInfo Development Guide

This document provides guidelines for agentic coding agents working on the PySInfo codebase.

## Project Overview

PySInfo is a Python command-line tool that displays system information including hardware and software details. It supports Windows, Linux, and macOS.

## Build, Lint, and Test Commands

### Installation

```bash
# Install dependencies
pip install -r requirements.txt

# Install in development mode
pip install -e .
```

### Running the Application

```bash
# Run as module (recommended)
python -m pysinfo

# Run directly
python run.py

# After installation, run as command
pysinfo
```

### Building for Distribution

```bash
# Create source distribution
python setup.py sdist

# Upload to PyPI (requires twine)
twine upload dist/*
```

### Testing

No test suite currently exists. When adding tests, use pytest:

```bash
# Run all tests
pytest

# Run a single test file
pytest tests/test_filename.py

# Run a single test function
pytest tests/test_filename.py::test_function_name

# Run with verbose output
pytest -v
```

### Linting and Type Checking

No linting configuration currently exists. Recommended tools:

```bash
# Format code with black
black pysinfo/

# Lint with ruff
ruff check pysinfo/

# Type check with mypy
mypy pysinfo/
```

## Code Style Guidelines

### Imports

Group imports in the following order, separated by blank lines:

1. Standard library imports (alphabetical)
2. Third-party imports (alphabetical)
3. Local imports

Example:
```python
import datetime
import os
import platform
import re
import shutil
import socket
import subprocess
import sys

import colorama
import distro
import GPUtil
import psutil
from colorama import Fore, Style, Back
```

### Naming Conventions

- **Functions**: `snake_case` (e.g., `get_system_info`, `check_cuda_version`)
- **Variables**: `snake_case` (e.g., `cpu_info`, `memory_info`)
- **Constants**: `UPPER_SNAKE_CASE` for module-level constants
- **Private functions**: Prefix with underscore (e.g., `_internal_helper`)

### String Formatting

Use f-strings for all string formatting:

```python
# Correct
info["os"] = f"{platform.system()} {platform.version()}"
print(f"CPU Model: {cpu_info['model']}")

# Avoid
info["os"] = "{} {}".format(platform.system(), platform.version())
```

### Docstrings

Use triple-quoted docstrings for functions:

```python
def get_detailed_cpu_info():
    """Get more detailed CPU information."""
    ...

def check_cuda_version():
    """
    Check the installed CUDA version.
    
    Returns:
        str: CUDA version string or "Not detected"
    """
    ...
```

### Error Handling

Use try-except blocks for operations that may fail. Prefer specific exceptions when possible, but broad exception handling is acceptable for system information gathering:

```python
# Acceptable for system info gathering
try:
    result = subprocess.run(["nvcc", "--version"], capture_output=True, text=True)
    if result.returncode == 0:
        ...
except:
    return "Not detected"

# Better - specify exception type
try:
    result = subprocess.run(["nvcc", "--version"], capture_output=True, text=True, timeout=5)
    ...
except (subprocess.SubprocessError, FileNotFoundError):
    return "Not detected"
```

Return sensible default values on failure (e.g., "Unknown", "Not detected", empty dict).

### Type Hints

Type hints are not currently used but are encouraged for new code:

```python
def get_system_info() -> dict:
    ...

def bytes_to_readable(bytes: int) -> str:
    ...

def get_ascii_logo(system: str) -> list[str]:
    ...
```

### Functions

- Keep functions focused on a single responsibility
- Return dictionaries for complex data structures
- Use early returns to reduce nesting
- Avoid deeply nested conditionals

### Comments

Use comments sparingly. Code should be self-documenting through clear naming. Use comments for:

- Section headers (e.g., `# OS Information`, `# Hardware Information`)
- Explaining non-obvious logic
- TODOs and FIXMEs

### Platform-Specific Code

The codebase handles Windows, Linux, and macOS. Use `platform.system()` to branch:

```python
if platform.system() == "Windows":
    # Windows-specific code
elif platform.system() == "Darwin":
    # macOS-specific code
else:
    # Linux-specific code (default)
```

### File Encoding

Use UTF-8 encoding for all files. When reading files that may have encoding issues:

```python
try:
    with open(filepath, encoding='utf-8') as f:
        content = f.read()
except UnicodeDecodeError:
    # Handle fallback
    ...
```

## Project Structure

```
Pysinfo/
├── setup.py              # Package configuration
├── requirements.txt      # Dependencies
├── run.py               # Standalone runner
├── pysinfo/
│   ├── __init__.py      # Main code (functions)
│   ├── __main__.py      # Entry point for `python -m pysinfo`
│   └── setup.py         # Duplicate of root setup.py
├── README.md            # Documentation
├── README_CN.md         # Chinese documentation
├── LICENSE              # MIT License
├── .gitignore           # Git ignore patterns
└── images/              # Example screenshots
```

## Dependencies

- **psutil**: System and process utilities
- **distro**: Linux distribution information
- **GPUtil-fix**: GPU information
- **colorama**: Terminal color output

## Key Functions

| Function | Purpose |
|----------|---------|
| `get_system_info()` | Main function that collects all system information |
| `print_system_info()` | Entry point that formats and displays output |
| `get_detailed_cpu_info()` | Collects CPU model, frequency, cache info |
| `get_detailed_memory_info()` | Collects RAM slots, speed, type info |
| `check_cuda_version()` | Detects CUDA installation |
| `check_opencl_version()` | Detects OpenCL support |
| `check_vulkan_version()` | Detects Vulkan support |
| `get_ascii_logo()` | Returns OS-specific ASCII art |
| `bytes_to_readable()` | Converts bytes to human-readable format |

## Common Tasks

### Adding a New System Info Field

1. Add the field to the dictionary in `get_system_info()`
2. Add the display line in `print_system_info()`'s `data_lines` list
3. Test on all supported platforms

### Adding Support for a New Platform

1. Add platform detection in relevant functions
2. Add platform-specific ASCII logo in `get_ascii_logo()`
3. Add color selection in `print_system_info()`

## Notes

- The main code lives in `pysinfo/__init__.py`
- `run.py` is a standalone version with the same code
- Be careful with subprocess timeouts on Windows
- GPU detection may fail silently on systems without NVIDIA GPUs

---
> Source: [EasyCam/Pysinfo](https://github.com/EasyCam/Pysinfo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
