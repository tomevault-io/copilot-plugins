## dagster-boilerplate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Dagster-based data engineering boilerplate project called "dagster-boilerplate" that provides a clean foundation for building data pipelines. The project is built with Python and uses PostgreSQL for data storage, with optional OpenAI integration for AI-powered features.

## Development Commands

### Package Management
- Install dependencies: `uv pip install -e .`
- Install with dev dependencies: `uv pip install -e ".[dev]"`

### Code Quality
- Format code: `black .`
- Sort imports: `isort .`
- Lint code: `flake8 .`
- Type checking: `mypy .`
- Run tests: `pytest`

### Dagster Operations
- Start Dagster dev server: `dagster dev`
- Migrate Dagster instance: `dagster instance migrate`
- Run specific assets: `dagster asset materialize --select sample_data_asset`

### Environment Setup
- Copy environment template: `cp .env.example .env`
- Required environment variables: `POSTGRES_HOST`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`
- Optional environment variables: `OUTPUT_PATH`, `MAX_FILE_SIZE_MB`, `OPENAI_API_KEY`

## Architecture

### Core Structure
- **data_pipeline/**: Main package with Dagster definitions
  - **assets/core/**: Data processing assets (includes `sample_data_asset` example)
  - **resources/**: Database and external service connections (PostgreSQL, OpenAI, file storage)
  - **jobs/**: Dagster job definitions (empty, to be populated)
  - **schedules/**: Automated scheduling (empty, to be populated)
  - **sensors/**: Event-driven triggers (empty, to be populated)
  - **utils/**: Helper functions for JSON parsing, numpy conversion, environment loading

### Key Resources
- **PostgreSQL**: Primary database with connection pooling and retry logic
- **OpenAI**: AI service integration via dagster-openai
- **File Storage**: Local file system storage with configurable paths and size limits

### Data Flow
- Sample data processing demonstrated in `sample_data_asset`
- Data directories: `data/raw/` for input, `data/processed/` for output
- Environment validation ensures required database credentials are present
- Flexible file storage with configurable paths and size limits

## Development Notes

- Uses `uv` package manager instead of pip
- Black and isort configured for 88-character line length
- MyPy configured for strict type checking
- The project structure follows Dagster best practices with clear separation of assets, resources, and utilities
- Environment variables are validated at startup to prevent runtime configuration errors
- This is a boilerplate project - customize assets, resources, and jobs for your specific use case
- Sample asset demonstrates basic logging and data processing patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kreshnik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kreshnik)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
