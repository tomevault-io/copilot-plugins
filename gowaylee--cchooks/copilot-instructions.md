## cchooks

> This file provides guidance for AI agents working with code in this repository.

# AGENT_GUIDE.md

This file provides guidance for AI agents working with code in this repository.

## Project Overview

This is a Python library (`cchooks`) for developing Claude Code hooks - user-defined shell commands that execute at various points in Claude Code's lifecycle. The library provides type-safe interfaces and utilities for all 8 hook types.

## Architecture

The codebase is organized into:

- **Core Types**: `src/cchooks/types.py` - Type definitions and literals for hook events, tools, and decisions
- **Base Classes**: `src/cchooks/contexts/base.py` - Abstract base classes for contexts and outputs
- **Hook Contexts**: Individual files in `src/cchooks/contexts/` for each hook type
- **Utilities**: `src/cchooks/utils.py` - JSON parsing and validation helpers
- **Exceptions**: `src/cchooks/exceptions.py` - Custom exception hierarchy

## Hook Types

1. **PreToolUse**: Runs before tool execution, can approve/block tools
2. **PostToolUse**: Runs after tool execution, can only provide feedback
3. **Notification**: Processes notifications, no decision control
4. **Stop**: Controls Claude's stopping behavior
5. **SubagentStop**: Controls subagent stopping behavior
6. **PreCompact**: Runs before transcript compaction
7. **UserPromptSubmit**: After user prompt submission
8. **SessionStart**: When Claude Code starts or resumes sessions
9. **SessionEnd**: When Claude Code session ends

## Essential Commands

### Testing
- `make test` - Run all tests with coverage
- `make test-quick` - Run tests without coverage (faster)
- `uv run pytest tests/contexts/test_pre_tool_use.py -v` - Run specific test file
- `uv run pytest tests/contexts/test_pre_tool_use.py::test_valid_context_creation -v` - Run single test

### Code Quality
- `make check` - Run all checks (lint, type-check, format, test)
- `make lint` - Lint code (ruff check)
- `make lint-fix` - Auto-fix linting issues
- `make format` - Format code (ruff format)
- `make type-check` - Type checking (mypy)

### Build & Setup
- `make setup` - Install dependencies
- `uv sync` - Alternative dependency install
- `make build` - Build package
- `make clean` - Clean build artifacts

## Code Style Guidelines

### Python Conventions
- **Type hints**: Required for all function signatures and public attributes
- **Imports**: Group imports (stdlib, third-party, local) with blank lines
- **Naming**:
  - Classes: PascalCase (PreToolUseContext)
  - Functions/Methods: snake_case (validate_fields)
  - Constants: UPPER_SNAKE_CASE (HOOK_TYPE_MAP)
  - Private: _single_underscore

### Error Handling
- Use custom exceptions from `cchooks.exceptions`
- Validate input data in context constructors
- Provide clear error messages with field names
- Use `HookValidationError` for missing required fields

### Testing Patterns
- Test files: `test_*.py` in `tests/` directory
- Test classes: `Test*`
- Test methods: `test_*`
- Use pytest fixtures for sample data
- Mock stdin/stdout for I/O testing

### Project Structure
- Main module: `src/cchooks/`
- Contexts: `src/cchooks/contexts/` (one file per hook type)
- Tests: Mirror source structure in `tests/`
- Type definitions: `src/cchooks/types.py`

### Hook Development
- Each hook has dedicated Context and Output classes
- Inherit from BaseHookContext/BaseHookOutput
- Validate required fields in constructor
- Use factory function `create_context()` for instantiation
- Follow JSON input/output patterns for Claude Code integration

## Input/Output Patterns
- **Simple Mode**: Exit codes (0=success, 1=non-blocking, 2=blocking)
- **Advanced Mode**: JSON with `continue`, `decision`, `reason` fields

## Usage Pattern

```python
from cchooks import create_context

# Read from stdin automatically
c = create_context()

# Type-specific handling
if isinstance(c, PreToolUseContext):
    if c.tool_name == "Write" and "password" in c.tool_input.get("file_path", ""):
        c.output.simple_block("Refusing to write to password file")
    else:
        c.output.simple_approve()
```

## Key Files

- `src/cchooks/__init__.py`: Main entry point and factory function
- `src/cchooks/contexts/pre_tool_use.py`: Most complex hook with approval decisions
- `src/cchooks/types.py`: Complete type system for Claude Code integration
- `docs/what-is-cc-hook.md`: Comprehensive documentation of Claude Code hooks

## Development Best Practices

- When generating git commit messages, follow patterns like "feat:", "fix:", "docs:", "refactor:" and other best practices
- Use type hints throughout the codebase
- Write tests for all new functionality
- Run `make check` before committing changes
- Follow existing naming conventions and code style
- Document public APIs with clear docstrings

## Project Structure

```
src/cchooks/
├── __init__.py           # Main factory function create_context()
├── types.py              # Type definitions and literals
├── exceptions.py         # Custom exception classes
├── utils.py              # JSON parsing utilities
└── contexts/
    ├── __init__.py       # Context exports
    ├── base.py          # Abstract base classes
    ├── pre_tool_use.py   # Pre-tool execution decisions
    ├── post_tool_use.py  # Post-tool execution feedback
    ├── notification.py   # Notification processing
    ├── stop.py          # Stop behavior control
    ├── subagent_stop.py  # Subagent stop control
    ├── pre_compact.py    # Pre-compaction processing
    ├── user_prompt_submit.py  # User prompt submission
    └── session_start.py  # Session start/resume

tests/
├── contexts/            # Context-specific tests
├── fixtures/            # Test data and helpers
├── integration/         # End-to-end tests
├── test_context_creation.py  # Factory function tests
├── test_exceptions.py   # Exception handling tests
├── test_types.py        # Type validation tests
└── test_utils.py        # Utility function tests
```
=== Content from CLAUDE.md (repaired 2025-08-04T10:54:24+08:00) ===
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python library (`cchooks`) for developing Claude Code hooks - user-defined shell commands that execute at various points in Claude Code's lifecycle. The library provides type-safe interfaces and utilities for all 8 hook types.

## Architecture

The codebase is organized into:

- **Core Types**: `src/cchooks/types.py` - Type definitions and literals for hook events, tools, and decisions
- **Base Classes**: `src/cchooks/contexts/base.py` - Abstract base classes for contexts and outputs
- **Hook Contexts**: Individual files in `src/cchooks/contexts/` for each hook type
- **Utilities**: `src/cchooks/utils.py` - JSON parsing and validation helpers
- **Exceptions**: `src/cchooks/exceptions.py` - Custom exception hierarchy

## Hook Types

1. **PreToolUse**: Runs before tool execution, can approve/block tools
2. **PostToolUse**: Runs after tool execution, can only provide feedback
3. **Notification**: Processes notifications, no decision control
4. **Stop**: Controls Claude's stopping behavior
5. **SubagentStop**: Controls subagent stopping behavior
6. **PreCompact**: Runs before transcript compaction
7. **UserPromptSubmit**: After user prompt submission
8. **SessionStart**: When Claude Code starts or resumes sessions
9. **SessionEnd**: When Claude Code session ends

## Development Commands

### Setup
```bash
# Install dependencies
make setup

# Or with uv directly
uv sync
```

### Testing
```bash
# Run all tests with coverage
make test

# Run tests without coverage (faster)
make test-quick

# Run specific test file
uv run pytest tests/contexts/test_pre_tool_use.py -v

# Run single test
uv run pytest tests/contexts/test_pre_tool_use.py::test_pre_tool_use_approve -v
```

### Linting and Formatting
```bash
# Check all code quality
make check

# Run individual checks
make lint          # ruff check
make type-check    # mypy
make format-check  # ruff format --check
make test          # pytest with coverage

# Auto-fix issues
make lint-fix      # ruff check --fix
make format        # ruff format
```

### Build and Distribution
```bash
# Build package
make build

# Clean build artifacts
make clean

# Full release preparation
make release-check
```

### Development Utilities
```bash
# Install in development mode
make install-dev

# Show dependency tree
make deps-tree

# Update lockfile
make lock
```

### Usage Pattern

```python
from cchooks import create_context

# Read from stdin automatically
c = create_context()

# Type-specific handling
if isinstance(c, PreToolUseContext):
    if c.tool_name == "Write" and "password" in c.tool_input.get("file_path", ""):
        c.output.simple_block("Refusing to write to password file")
    else:
        c.output.simple_approve()
```

## Input/Output Patterns

### Simple Mode (Exit Codes)
- `exit 0`: Success/approve
- `exit 1`: Non-blocking error
- `exit 2`: Blocking error

### Advanced Mode (JSON)
- Use context-specific output methods
- Each context provides specialized decision methods
- JSON output includes `continue`, `decision`, `reason` fields

## Project Structure

```
src/cchooks/
├── __init__.py           # Main factory function create_context()
├── types.py              # Type definitions and literals
├── exceptions.py         # Custom exception classes
├── utils.py              # JSON parsing utilities
└── contexts/
    ├── __init__.py       # Context exports
    ├── base.py          # Abstract base classes
    ├── pre_tool_use.py   # Pre-tool execution decisions
    ├── post_tool_use.py  # Post-tool execution feedback
    ├── notification.py   # Notification processing
    ├── stop.py          # Stop behavior control
    ├── subagent_stop.py  # Subagent stop control
    └── pre_compact.py    # Pre-compaction processing

tests/
├── contexts/            # Context-specific tests
├── fixtures/            # Test data and helpers
├── integration/         # End-to-end tests
├── test_context_creation.py  # Factory function tests
├── test_exceptions.py   # Exception handling tests
├── test_types.py        # Type validation tests
└── test_utils.py        # Utility function tests
```

## Key Files to Understand

- `src/cchooks/__init__.py`: Main entry point and factory function
- `src/cchooks/contexts/pre_tool_use.py`: Most complex hook with approval decisions
- `src/cchooks/types.py`: Complete type system for Claude Code integration
- `docs/what-is-cc-hook.md`: Comprehensive documentation of Claude Code hooks

## Development Best Practices

- When generating git commit messages, follow patterns like "feat:", "fix:", "docs:", "refactor:" and other best practices
- Use type hints throughout the codebase
- Write tests for all new functionality
- Run `make check` before committing changes
- Follow existing naming conventions and code style
- Document public APIs with clear docstrings
=== End CLAUDE.md content ===

=== Content from CRUSH.md (repaired 2025-08-04T10:54:24+08:00) ===
# cchooks Development Guide

## Project Overview
Python library for developing Claude Code hooks - user-defined shell commands that execute at various points in Claude Code's lifecycle. Provides type-safe interfaces for 9 hook types.

## Essential Commands

### Testing
- `make test` - Run all tests with coverage
- `make test-quick` - Run tests without coverage (faster)
- `uv run pytest tests/contexts/test_pre_tool_use.py -v` - Run specific test file
- `uv run pytest tests/contexts/test_pre_tool_use.py::test_valid_context_creation -v` - Run single test

### Code Quality
- `make check` - Run all checks (lint, type-check, format, test)
- `make lint` - Lint code (ruff check)
- `make lint-fix` - Auto-fix linting issues
- `make format` - Format code (ruff format)
- `make type-check` - Type checking (mypy)

### Build & Setup
- `make setup` - Install dependencies
- `uv sync` - Alternative dependency install
- `make build` - Build package
- `make clean` - Clean build artifacts

## Hook Types & Usage

### 9 Hook Types
1. **PreToolUse**: Before tool execution, can approve/block
2. **PostToolUse**: After tool execution, feedback only
3. **Notification**: Process notifications, no decisions
4. **Stop**: Control Claude stopping behavior
5. **SubagentStop**: Control subagent stopping
6. **PreCompact**: Before transcript compaction
7. **UserPromptSubmit**: After user prompt submission
8. **SessionStart**: When Claude Code starts or resumes sessions
9. **SessionEnd**: When Claude Code session ends

### Usage Pattern
```python
from cchooks import create_context

c = create_context()

if isinstance(c, PreToolUseContext):
    if c.tool_name == "Write" and "password" in c.tool_input.get("file_path", ""):
        c.output.simple_block("Refusing to write to password file")
    else:
        c.output.simple_approve()
```

## Code Style Guidelines

### Python Conventions
- **Type hints**: Required for all function signatures and public attributes
- **Imports**: Group imports (stdlib, third-party, local) with blank lines
- **Naming**:
  - Classes: PascalCase (PreToolUseContext)
  - Functions/Methods: snake_case (validate_fields)
  - Constants: UPPER_SNAKE_CASE (HOOK_TYPE_MAP)
  - Private: _single_underscore

### Error Handling
- Use custom exceptions from `cchooks.exceptions`
- Validate input data in context constructors
- Provide clear error messages with field names
- Use `HookValidationError` for missing required fields

### Testing Patterns
- Test files: `test_*.py` in `tests/` directory
- Test classes: `Test*`
- Test methods: `test_*`
- Use pytest fixtures for sample data
- Mock stdin/stdout for I/O testing

### Project Structure
- Main module: `src/cchooks/`
- Contexts: `src/cchooks/contexts/` (one file per hook type)
- Tests: Mirror source structure in `tests/`
- Type definitions: `src/cchooks/types.py`

### Hook Development
- Each hook has dedicated Context and Output classes
- Inherit from BaseHookContext/BaseHookOutput
- Validate required fields in constructor
- Use factory function `create_context()` for instantiation
- Follow JSON input/output patterns for Claude Code integration

## Input/Output Patterns
- **Simple Mode**: Exit codes (0=success, 1=non-blocking, 2=blocking)
- **Advanced Mode**: JSON with `continue`, `decision`, `reason` fields

## Key Files
- `src/cchooks/__init__.py`: Main entry point and factory function
- `src/cchooks/contexts/pre_tool_use.py`: Most complex hook with approval decisions
- `src/cchooks/types.py`: Complete type system for Claude Code integration
=== End CRUSH.md content ===

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GowayLee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GowayLee)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
