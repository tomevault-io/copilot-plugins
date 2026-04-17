## dateutils-python

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development Environment Setup
- `make dev` - Set up complete development environment (deps, install, pre-commit hooks)
- `make deps` - Install development dependencies using uv
- `make install` - Install the package in development mode

### Testing
- `make test` - Run full test suite with coverage
- `make test-fast` - Run tests without coverage for faster feedback
- `make coverage` - Run tests with coverage report
- `make coverage-html` - Generate HTML coverage report

### Code Quality
- `make lint` - Run ruff linting checks
- `make lint-fix` - Run ruff linting and fix auto-fixable issues
- `make format` - Format code using ruff
- `make format-check` - Check if code is properly formatted
- `make typecheck` - Run ty static type checking
- `make check` - Run all checks (lint, format-check, typecheck, test)
- `make fix` - Fix common issues (format + lint-fix)

### Build and Distribution
- `make build` - Build the package for distribution
- `make build-check` - Build and check the package with twine

### Versioning
- `make version-patch` - Bump patch version and push tag
- `make version-minor` - Bump minor version and push tag
- `make version-major` - Bump major version and push tag

### Utilities
- `make clean` - Remove build artifacts and cache files
- `make version` - Show current version information

## Architecture

### Core Structure
- **Single Module Design**: All functionality is contained in `dateutils/dateutils.py` - a single, comprehensive module
- **Standard Library Only**: No external dependencies beyond Python's standard library
- **Full Type Annotations**: Complete type hints using Python 3.9+ typing features
- **Comprehensive Testing**: Single test file `tests/test_dateutils.py` with extensive coverage using pytest and freezegun

### Key Components
- **UTC Operations**: Core timestamp and UTC date functions (`utc_now_seconds`, `utc_today`, `epoch_s`)
- **Business Day Logic**: Weekend and holiday-aware calculations (`workdays_between`, `add_business_days`)
- **US Federal Holidays**: Built-in cached holiday calculations for business day functions
- **Timezone Handling**: Robust timezone conversion using `zoneinfo`
- **Date Parsing**: Multi-format date/datetime parsing with comprehensive validation
- **Quarter/Month/Week Operations**: Time period calculations and generators
- **Human-readable Formatting**: Pretty date formatting ("2 hours ago", "Yesterday")

### Module Organization
- All public functions are re-exported through `dateutils/__init__.py`
- Constants for common time calculations (SECONDS_IN_DAY, DAYS_IN_WEEK, etc.)
- Extensive input validation with descriptive error messages
- Generator-based functions for memory-efficient date range operations
- LRU caching for expensive operations like holiday calculations

### Development Workflow
- Uses `uv` for fast dependency management
- `ruff` for linting and formatting (configured for 120 character line length)
- `ty` for fast static type checking (Astral's Rust-based type checker)
- `pytest` with `freezegun` for time-sensitive testing
- Pre-commit hooks for code quality enforcement
- Comprehensive Makefile for all development tasks

### Testing Strategy
- Single comprehensive test file covering all functionality
- Time-based testing using `freezegun` for consistent results
- Coverage requirements enforced through pytest configuration
- Tests validate both success cases and error conditions with proper exception handling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chetmancini) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
