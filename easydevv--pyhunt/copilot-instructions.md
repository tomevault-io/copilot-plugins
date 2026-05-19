## pyhunt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`pyhunt` is a lightweight Python logging tool that automatically traces function calls and provides visual, tree-structured colored logs for debugging. It uses a simple `@trace` decorator to log function execution times, call hierarchies, and exceptions.

## Development Commands

### Package Management
```bash
# Install dependencies
uv sync

# Build the package
uv build
```

### Python Execution
```bash
# Run Python files with uv
uv run python filename.py

# Run Python modules with uv
uv run python -m module_name

# Run Python interactively with uv
uv run python
```


### Testing
```bash
# Run all tests
uv run pytest

# Run specific test file
uv run pytest test/test_examples.py

# Run tests with verbose output
uv run pytest -v
```

### Code Quality
```bash
# Lint all files in the current directory
uvx ruff check

# Format all files in the current directory
uvx ruff format
```

### CLI Tool
```bash
# Set up environment file (creates .env with DEBUG level)
hunt

# Set specific log levels
hunt --debug
hunt --info
hunt --warning
hunt --error
hunt --critical

# Set root directory and repeat limit
hunt --root
hunt --repeat 5
```


## Core Architecture

### Main Components
- **`pyhunt/decorator.py`**: Core `@trace` decorator implementation for function call tracing
- **`pyhunt/cli.py`**: Command-line interface for managing log levels and environment variables
- **`pyhunt/logger.py`**: Direct logging interface for manual log messages
- **`pyhunt/console.py`**: Console output formatting and color handling
- **`pyhunt/config.py`**: Environment variable configuration and `.env` file management
- **`pyhunt/colors.py`**: Color definitions for different log levels
- **`pyhunt/context.py`**: Call context management for tracking execution depth
- **`pyhunt/utils.py`**: Utility functions for path handling and formatting
- **`pyhunt/helpers.py`**: Helper functions for decorator and logging operations

### Key Features
- **Automatic Tracing**: The `@trace` decorator automatically logs function entry, exit, execution time, and exceptions
- **Tree-structured Output**: Logs are indented based on call depth for visual hierarchy
- **Multiple Log Levels**: DEBUG, INFO, WARNING, ERROR, CRITICAL with color coding
- **Async Support**: Full support for async/await functions
- **Class Integration**: Can be applied to entire classes to trace all methods
- **Environment Configuration**: Manages log levels through `.env` file and CLI

## Usage Patterns

### Basic Usage
```python
from pyhunt import trace

@trace
def my_function(param):
    return param * 2

@trace
async def async_function(param):
    return await some_async_call()

@trace
class MyClass:
    def method_one(self):
        pass
    
    def method_two(self):
        pass
```

### Direct Logging
```python
from pyhunt import logger

logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
logger.critical("Critical message")
```

## Configuration

The tool supports these environment variables (managed via `.env` file):
- `HUNT_LEVEL`: Log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
- `HUNT_MAX_REPEAT`: Maximum number of repeated log displays
- `ELAPSED`: Whether to show execution time (True/False)
- `ROOT_DIR`: Base directory for path display

## AI Integration Rules

For AI-generated code, follow these patterns:
- Import: `from pyhunt import trace`
- Use `@trace` decorator instead of `print()` statements
- For exceptions, use `try/except Exception as e: raise e` to maintain traceback
- Apply `@trace` to classes and methods automatically when adding logging
- Execute Python files with `uv run python` instead of just `python`

## Testing

Tests are located in the `test/` directory and use pytest. The main test file `test_examples.py` runs all example files and compares their output against expected results stored in `examples/outputs/`.

---
> Source: [EasyDevv/pyhunt](https://github.com/EasyDevv/pyhunt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
