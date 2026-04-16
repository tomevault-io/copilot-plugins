## vitest-create-mock

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Vitest Toolbox**, a port of [@golevelup/ts-jest](https://github.com/golevelup/nestjs/tree/master/packages/testing/ts-jest), specifically providing a counterpart of `createMock` for Vitest.

## Package Manager

This project uses **pnpm** (not npm or yarn). Always use `pnpm` commands.

## Common Commands

### Development
- `pnpm run dev` - Watch mode for development (runs tsdown with --watch)
- `pnpm run build` - Build the library using tsdown
- `pnpm run typecheck` - Type check without emitting files

### Testing
- `pnpm run test` - Run all tests with Vitest
- `pnpm run test -- path/to/test.test.ts` - Run a single test file

### Release
- `pnpm run release` - Bump version and publish to npm

## Architecture

### Build System
The project uses **tsdown** (not tsc or other bundlers) for building. Configuration is in [tsdown.config.ts](tsdown.config.ts):
- Entry point: [src/index.ts](src/index.ts)
- Platform: neutral
- Outputs to `dist/` with TypeScript declarations

### Source Structure
- Source code: [src/](src/) directory
- Tests: [tests/](tests/) directory (`.test.ts` extension)
- Single entry point at [src/index.ts](src/index.ts)

### TypeScript Configuration
Uses strict TypeScript with:
- Target: esnext, lib: es2023
- Module: preserve (bundler resolution)
- Strict mode enabled with noUnusedLocals
- Declaration files emitted via tsconfig, bundling via tsdown

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jakeklassen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
