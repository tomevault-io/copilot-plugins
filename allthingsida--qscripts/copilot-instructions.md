## qscripts

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# qscripts_native - IDA Pro Plugin Template

This is a template for developing IDA Pro plugins using the QScripts hot-reload workflow. The plugin lists non-library functions in the current database.

## Build Commands

Always use the ida-cmake agent for building:
```
Task with subagent_type="ida-cmake"
```

## Prerequisites

- **IDASDK**: Environment variable must be set to IDA SDK path
- **IDAX**: Must be installed at `$IDASDK/include/idax/`
- **ida-cmake**: Must be installed at `$IDASDK/ida-cmake/`

## Architecture

### File Structure
- `driver.cpp`: IDA plugin interface boilerplate (PLUGIN structure and plugmod_t implementation)
- `main.cpp`: Core plugin logic - implement your functionality here
- `idasdk.h`: Consolidated IDA SDK header includes with warning suppression
- `CMakeLists.txt`: Build configuration using ida-cmake

### Plugin Behavior
- **Type**: PLUGIN_UNL | PLUGIN_MULTI (unloadable, multi-instance)
- **Entry Point**: `main()` function in main.cpp receives size_t argument
- **Current Function**: Enumerates all non-library functions and prints their names

## Development Workflow

This template is designed for use with QScripts hot-reload:
1. Parent directory contains `qscripts_native.py.deps.qscripts` with trigger file configuration
2. Built plugin outputs to `$IDASDK/bin/plugins/qscripts_native[.dll/.so/.dylib]`
3. QScripts monitors the output file and automatically reloads on changes

## SDK and IDA API Resources

When answering SDK/API questions, search and read from:
- **SDK Headers**: `$IDASDK/include` - All headers have docstrings
- **SDK Examples**: `$IDASDK/plugins`, `$IDASDK/loaders`, `$IDASDK/module`

---
> Source: [allthingsida/qscripts](https://github.com/allthingsida/qscripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
