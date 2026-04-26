## ocf-rules

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OCF-Tools is a TypeScript library for validating and working with [Open Cap Table Format (OCF)](https://open-cap-table-coalition.github.io/Open-Cap-Format-OCF/) datasets. OCF is a standard for representing cap table data (stakeholders, stock classes, transactions, vesting terms, valuations). The library is pre-NPM-publication and consumed via `npm link`.

## Commands

- **Build:** `npm run build` (runs `tsc`)
- **Test (watch mode):** `npm test` (runs `jest --watch`)
- **Run single test:** `npx jest path/to/test.test.ts`
- **Print examples:** `npm run print:vesting_schedule`, `npm run print:vesting_status`, `npm run print:iso_calculator`
- **Node version:** v20 (see `.nvmrc`)

## Architecture

### Five Tools

1. **`read_ocf_package/`** - Reads an OCF folder from disk into an `OcfPackageContent` object by parsing `Manifest.ocf.json` and all referenced files. This is the entry point for all other tools.

2. **`vesting_schedule_generator_v1/`** (preferred) - Graph-based vesting schedule generator. Models vesting conditions as a directed acyclic graph (DAG), then walks an execution path to produce installments.
   - `VestingScheduleGenerator` is instantiated with `OcfPackageContent`, `ExecutionPathBuilder`, and `VestingConditionStrategyFactory`
   - Three strategies: imperative (vesting_terms_id), declarative (vestings array), fully-vested (neither)
   - Uses Strategy pattern via `VestingConditionStrategyFactory` to handle trigger types: `VESTING_START_DATE`, `VESTING_EVENT`, `VESTING_SCHEDULE_ABSOLUTE`, `VESTING_SCHEDULE_RELATIVE`
   - `VestingModeService` handles allocation types (CUMULATIVE_ROUNDING, FRONT_LOADED, BACK_LOADED, etc.)

3. **`vesting_schedule_generator/`** (alternative) - Older implementation using `VestingScheduleService`. Uses a non-standard `cliff_length` field on `VestingRelativeTrigger` not yet in the OCF spec.

4. **`iso_nso_calculator/`** - Calculates ISO/NSO splits per the $100K annual capacity rule. Uses `vesting_schedule_generator_v1` internally. Relies on a non-standard `valuation_id` field on equity compensation issuances.

5. **`ocf_validator/`** - XState state machine that validates OCF packages by processing sorted transactions day-by-day with end-of-day (EOD) checks. Validators are organized by transaction category (stock, equity_compensation, convertible, warrant, plan_security, vesting, stock_class, stock_plan).

6. **`ocf_snapshot/`** - Returns the cap table state as of a given date by filtering validator snapshots.

### Key Patterns

- **Types are defined locally** in each generator's `types/index.ts`, mirroring OCF schema types (not imported from OCF). The two generators have parallel but slightly different type definitions (e.g., `cliff_installment` vs `cliff_length`).
- **`OcfPackageContent`** (defined in `read_ocf_package/index.ts`) is the shared data structure passed between tools.
- **Jest module alias:** `vesting_schedule_generator` is aliased to `vesting_schedule_generator_v1` in `jest.config.js` via `moduleNameMapper`.
- **Test data** lives in `sample_ocf_folders/` (real OCF packages) and `vesting_schedule_generator_v1/tests/testOcfPackages/` (programmatic builders).
- **Testing scripts** in `testing_scripts/` are runnable via `ts-node` for manual verification.

### Dependencies

- `date-fns` for date arithmetic
- `fraction.js` for precise fractional share math
- `xstate` for the validator state machine
- `tsconfig-paths` for path resolution in ts-node scripts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JSv4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
