## multi-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DataFlowPipeline is a Python framework for building data transformation pipelines with validation, transformation, error handling, and rollback capabilities. It implements a stage-based architecture where records flow through configurable pipeline stages.

## Architecture

### Core Abstractions

**Pipeline Flow**: `Pipeline` → `Stage` (Validator/Transform/Loader) → `Record`

The framework is built around three key abstractions:

1. **Pipeline** (`pipeline.py`): Orchestrates execution of records through multiple stages
   - Supports both sequential and parallel processing (`parallel=True`)
   - Optional error handling with automatic or custom error handlers
   - Tracks metrics: processed count, stage-level pass rates
   - `process_record()` processes single records through all stages sequentially
   - `process_batch()` processes multiple records (uses `ThreadPoolExecutor` if parallel=True)

2. **Stages** (`stages.py`): Abstract base class with concrete implementations
   - `ValidatorStage`: Validates records using one or more `Validator` instances
   - `TransformStage`: Transforms records using one or more `Transform` instances
   - `LoaderStage`: Persists records to storage (tracks loaded records in memory)
   - All stages track `_passed_count` and support optional `rollback()`

3. **Error Handling** (`handlers.py`): Pluggable error recovery strategies
   - `ErrorHandler`: Base class that logs errors and stops processing
   - `RollbackHandler`: Calls `rollback()` on all stages when errors occur
   - `RetryHandler`: Retries failed records up to `max_retries` times (default: 3)

### Supporting Components

**Validators** (`validators.py`): Return data if valid, `None` if invalid
- `TypeValidator`: Validates required fields have correct types
- `RangeValidator`: Validates numeric fields are within min/max bounds
- `SchemaValidator`: Validates schema version (warns on mismatch)

**Transforms** (`transforms.py`): Modify data and return transformed dict
- `ScaleTransform`: Multiplies numeric field by factor (handles `Decimal` conversion)
- `FilterTransform`: Marks records with `_filtered=True` if value not in allowed list
- `DivisionTransform`: Divides numeric field by divisor (raises on division by zero)
- `CurrencyConversionTransform`: Converts currency amounts by exchange rate

**Models** (`models.py`): Dataclass-based record structures
- `Record`: Base record with `record_id`, `data`, and `metadata` dicts
- `FinancialRecord`: Specialized record with `amount`, `tax_amount`, `currency`
  - Auto-converts float/Decimal to int for amount fields in `__setattr__`
  - `apply_tax_rate()` calculates tax from amount
  - `from_dict()` factory method for deserialization

## Key Design Patterns

### Rollback Support

Stages implement `rollback()` to undo operations:
- `ValidatorStage`: Clears `_validation_failures` and resets `_passed_count`
- `TransformStage`: Clears `_transformed_records` and resets `_passed_count`
- `LoaderStage`: Does NOT implement rollback (inherits warning-only base implementation)

**Rollback Caveat**: `LoaderStage` does not clear `_loaded_records`, meaning rollback is incomplete for persistence operations.

---
> Source: [religa/multi_mcp](https://github.com/religa/multi_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
