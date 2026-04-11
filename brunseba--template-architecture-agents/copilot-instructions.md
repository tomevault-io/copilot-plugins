## template-architecture-agents

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Documentation Tracking System - A Python CLI tool (`track`) for tracking input-output mappings between source files and generated documentation. Validates file synchronization using SHA-256 hashes and maintains history with git commit integration.

## Build & Development Commands

```bash
# Setup dev environment (installs deps + pre-commit hooks)
make dev

# Run tests with coverage
make test
# or fast-fail mode
pytest tests/ -v -x

# Run single test
pytest tests/test_tracking.py::TestModels::test_mapping_creation -v

# Linting and formatting
make lint
make format

# Validate tracking file and sync
make validate
make sync

# All CI checks locally
make ci
```

## Architecture

### Core Components (`src/tracking_manager/`)

- **`cli.py`** - Click-based CLI commands (`track`, `validate`, `sync`, `status`, `history`, `migrate`)
- **`models.py`** - Pydantic models with backward compatibility validators:
  - `Mapping` - Multi-file input/output mapping with hash tracking (supports old single-file format via `model_validator`)
  - `Generation` - Documentation generation metadata
  - `TrackingData` - Root model containing version, last_generation, history
- **`utils.py`** - File operations: git integration, SHA-256 hashing, JSON schema validation, sync checking

### Data Flow

1. Input files (`inputs/`) → Transformation → Output docs (`docs/`)
2. Mappings tracked in `.docs-tracking.json` with file hashes
3. Pre-commit hooks validate tracking file and check file sync

### Key Files

- `.docs-tracking.json` - Main tracking data (validated against `schema/tracking-schema.json`)
- `scripts/` - Standalone validation scripts for pre-commit hooks
- `templates/` - Draw.io and ArchiMate templates (see `opengroup-archimate-library.xml`)
- `naming-and-standards/` - Reference guides for formats (ArchiMate, Draw.io, JSON Schema, Conventional Commits, SemVer)

## Testing

Tests use pytest with fixtures from `tmp_path`. Test classes:
- `TestModels` - Pydantic model validation including backward compatibility
- `TestUtils` - File hashing, consistency checks
- `TestTrackingFile` - JSON structure, schema validation
- `TestIntegrity` - History order, duplicate detection

## CLI Entry Point

The CLI is registered as `track` command (via `pyproject.toml [project.scripts]`). Examples:

```bash
track status                    # Show tracking status
track track input.csv -o out.md -d "Description" --with-hash
track migrate                   # Migrate old single-file format to multi-file
```

## Draw.io Library (ArchiMate)

Templates use custom UserObject properties accessible via Edit Data (Ctrl+M):
- `Layer`, `Component`, `ApplicationName`, `EnvironmentName`

ArchiMate layer colors: Motivation (#CCCCFF), Strategy (#F5DEAA), Business (#FFFFB5), Application (#B5FFFF), Technology (#C9E7B7), Implementation (#FFE0E0), External (#f5f5f5)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brunseba)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brunseba)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
