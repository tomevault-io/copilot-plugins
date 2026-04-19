## dotnet-test-shard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitHub Action for automatic .NET test sharding. It provides Playwright-style test sharding for .NET test projects without requiring manual test annotations.

**Algorithm:** Deterministic modulo-based distribution
1. Discover all tests via `dotnet test --list-tests`
2. Sort tests alphabetically for consistency
3. Assign test[i] to shard = (i % total_shards) + 1
4. Generate filter expression using `FullyQualifiedName~`

## Commands

```bash
# Install dependencies
npm ci

# Run all tests
npm test

# Run tests in watch mode
npm test:watch

# Run tests with coverage
npm test -- --coverage

# Run a specific test file
npm test -- __tests__/filter.test.ts

# Run tests matching a pattern
npm test -- --testNamePattern="getTestsForShard"

# Build the action (outputs to dist/)
npm run build

# Lint code
npm run lint

# Format code
npm run format

# Run all checks (format, lint, test, build)
npm run all
```

## Architecture

### Source Files (`src/`)

- **index.ts** - Main entry point. Orchestrates: get inputs → validate → discover tests → shard → run tests → set outputs
- **types.ts** - TypeScript interfaces: `ActionInputs`, `ActionOutputs`, `DiscoveryResult`, `TestRunResult`
- **discover.ts** - Test discovery via `dotnet test --list-tests`. Parses output, strips Theory parameters, deduplicates
- **filter.ts** - Sharding logic. `getTestsForShard()` uses modulo distribution. `buildFilterExpression()` generates VSTest filter
- **runner.ts** - Executes `dotnet test` with filter, aggregates results from multiple TRX files (one per assembly)

### Test Mocks (`__tests__/mocks/`)

Jest mocks for `@actions/core` and `@actions/exec` packages. Configured in `jest.config.js` via `moduleNameMapper`.

### Build Output

The action runs from `dist/index.js` (bundled via `@vercel/ncc`). The pre-commit hook rebuilds and verifies dist/ is up-to-date.

## Key Patterns

- **Sharding formula:** `test[i]` goes to shard `(i % totalShards) + 1` (1-based shards)
- **Filter syntax:** Uses `FullyQualifiedName~TestName` (contains operator) to match Theory variants
- **TRX aggregation:** Dotnet generates unique TRX files per assembly when LogFileName is not specified
  - `runner.ts` aggregates results from all `*.trx` files in the results directory
  - Each assembly creates a timestamped file (e.g., `TestResults_user_2024-01-25_08_13_03.trx`)
  - Prevents TRX file overwrites when testing solutions with multiple assemblies
- **TRX parsing:** Regex-based extraction of `<Counters total="X" passed="Y" failed="Z" executed="W" notExecuted="V"/>` attributes
  - `notExecuted` is used for accurate skipped test counts (fixes issue where `executed - passed - failed` always equals 0)
  - Falls back to `total - executed` when `notExecuted` attribute is missing
  - The `resultFile` output now returns the directory path containing all TRX files, not a single file path

## Theory Test Handling

xUnit/NUnit Theory tests (parameterized tests) produce multiple test variants from a single test method:
- **Discovery output:** `MyNamespace.MyClass.TestMethod` (base name only)
- **Actual execution:** `MyNamespace.MyClass.TestMethod(param1)`, `MyNamespace.MyClass.TestMethod(param2)`, etc.

The action handles this by:
1. Deduplicating Theory variants during discovery (strips parameters)
2. Using `FullyQualifiedName~BaseName` in filters (contains operator `~` matches all variants)

This ensures all Theory variants stay together in the same shard.

## Testing the Action Locally

The `test-project/` directory contains a sample .NET test project with ~30 xUnit tests for integration testing.

```bash
# Run the test project directly
cd test-project
dotnet test

# List tests (what the action does for discovery)
dotnet test --list-tests
```

## Important Notes

- Node.js 22 required (see `.nvmrc`)
- Pre-commit hook runs build and checks for uncommitted dist/ changes
  - If dist/ has changes after commit, you must stage them: `git add dist/`
- CI verifies dist/ is up-to-date via `verify-dist` job
- This is a GitHub Action - changes to `src/` must be compiled to `dist/` via `npm run build`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WonderForgeLabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
