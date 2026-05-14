## electron-async-storage

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `electron-async-storage`, a TypeScript library providing an async storage interface for Electron applications based on the unstorage architecture. The library follows a driver-based pattern similar to unstorage, with built-in support for filesystem and memory storage drivers.

## Core Architecture

- **Storage Core** (`src/storage.ts`): Main storage abstraction with mount-based driver system
- **Driver System** (`src/drivers/`): Pluggable storage drivers (fs, fs-lite, memory, queue)
- **Type System** (`src/types.ts`): Complete TypeScript interfaces for Storage, Driver, and related types
- **Build System**: Auto-generates driver exports using `scripts/gen-drivers.ts`

## Key Concepts

- **Mountpoints**: Storage can mount different drivers at different base paths
- **Driver Interface**: All drivers implement the same `Driver` interface with methods like `getItem`, `setItem`, `getKeys`
- **Serialization**: Uses `superjson` for complex object serialization
- **Migrations**: Built-in version-based migration system with hooks
- **Watching**: File system watching support for real-time updates

## Development Commands

```bash
# Build the library (generates drivers and compiles)
pnpm build

# Run development with tests in watch mode
pnpm dev

# Run all tests with coverage
pnpm test

# Run type checking only
pnpm test:types

# Lint and format
pnpm lint
pnpm lint:fix

# Generate driver exports (auto-updates src/_drivers.ts)
pnpm gen-drivers
```

## Testing

- Uses Vitest as the test runner
- Tests are in `test/` directory, organized by functionality
- Coverage excludes `drivers/` and `scripts/` directories
- Type checking is enabled in test configuration

## Build Process

1. `gen-drivers` - Auto-generates driver type definitions and exports
2. `unbuild` - Compiles TypeScript to multiple formats (ESM, CJS)
3. `fix-driver-imports` - Post-processes driver import paths

The build outputs both main library files and individual driver files to support tree-shaking.

---
> Source: [YougLin-dev/electron-async-storage](https://github.com/YougLin-dev/electron-async-storage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
