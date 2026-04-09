## set-object-utils

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TypeScript utility library extending native Map with Set-like operations. Uses Bun runtime/test runner and Biome for linting/formatting.

## Development Commands

```bash
# Testing
bun test                    # Run all tests
bun test:watch             # Run tests in watch mode
bun test test/[file].test.ts  # Run specific test file

# Code Quality
bun run lint               # Check code with Biome
bun run lint:fix          # Fix linting issues
bun run format            # Format code with Biome
bun run typecheck         # TypeScript type checking (no emit)

# Development
bun run example           # Run index.ts
bun run benchmark         # Run all benchmarks
bun run benchmark/[file].bench.ts  # Run specific benchmark
```

## Architecture

### Hybrid Design Pattern
The codebase uses a hybrid approach combining class inheritance with standalone functions:
- **ExtendedMap class** (`src/extended-map.ts`): Extends native Map, delegates to standalone functions
- **Standalone functions** (`src/*.ts`): Pure functions that operate on Map instances, enabling tree-shaking
- This pattern provides both OOP convenience and functional programming flexibility

### Module Structure
- Each Set operation is implemented in its own module for tree-shaking
- All functions return `ExtendedMap` instances to enable method chaining
- Type definitions in `src/types.ts` ensure consistency

### Key Implementation Details
- **Default values**: ExtendedMap constructor accepts options with default value support
- **Value equality**: Set operations compare both keys AND values (not just keys)
- **Performance**: Operations use native Map iteration for optimal performance
- **Type safety**: Strict TypeScript with `noUncheckedIndexedAccess` enabled

## Testing Strategy
- Each operation has dedicated test file in `test/` directory
- Tests cover edge cases: empty Maps, single elements, value mismatches
- Default value behavior is tested in `extended-map.test.ts`

## Benchmarking
- Performance benchmarks in `benchmark/` directory
- Utilities in `benchmark/utils.ts` for consistent test data generation
- Benchmarks test various Map sizes (10, 1000, 100000 items)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuyakinjo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yuyakinjo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
