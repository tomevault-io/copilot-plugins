## jsdom-extended

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`jsdom-extended` is a TypeScript library that provides browser API mocks for JSDOM testing environments. It patches missing or incomplete browser features like geometry calculations, requestAnimationFrame, ResizeObserver, and location helpers to enable realistic UI component testing.

## Build and Development Commands

```bash
# Type checking only
yarn types:check

# Full build (type check + vite build)
yarn build

# Watch mode for development
yarn build:watch
```

The build process uses Vite and generates multiple output formats:
- ESM: `dist/jsdom-extended.js`
- UMD: `dist/jsdom-extended.umd.cjs` (minified and non-minified)
- Type definitions: `dist/jsdom-extended.d.ts` and `dist/jsdom-extended.d.cts`

## Architecture

### Core Pattern: Window Patching
The library works by directly patching the JSDOM window object's prototype chain and global properties. Each mock is implemented as an independent patch function that modifies specific browser APIs.

### Module Structure
```
src/
├── index.ts              # Public API exports
├── setup.ts              # applyJsdomExtended - orchestrates all patches
└── mocks/
    ├── geometry.ts       # Patches getBoundingClientRect and window dimensions
    ├── raf.ts            # Implements requestAnimationFrame/cancelAnimationFrame
    ├── resize-observer.ts # FakeResizeObserver class
    └── location.ts       # Location validation and navigation helpers
```

### Patch Function Pattern
Each mock module exports a `patch*` function that accepts `window: Window & typeof globalThis` and uses `Object.defineProperty` to install mocks. This ensures compatibility with JSDOM's property model while allowing test code to override values if needed.

**Important:** The geometry mock provides fixed values (100x50 for elements, 1280x720 for window) that all tests depend on. Changes to these values would break consumer tests.

### RAF Implementation Detail
The `patchRAF` function in `src/mocks/raf.ts` uses a `Map<number, NodeJS.Timeout>` to track active animation frame handles. This allows proper cancellation via `cancelAnimationFrame`. The ~60fps simulation uses a 16ms setTimeout delay.

### Location Helpers Philosophy
Unlike other mocks, `patchLocation` doesn't modify `window.location` (JSDOM already provides full support). Instead, it validates the location is properly configured and logs warnings if the URL is `about:blank`. The helper functions (`hasValidLocation`, `navigateTo`, `getLocationInfo`) are utility wrappers for test assertions.

## TypeScript Configuration

- Target: ESNext with ES2020 lib
- Strict mode enabled with all strict flags
- Composite project with declaration maps for downstream consumers
- `verbatimModuleSyntax: true` - imports must explicitly use type-only imports where applicable

## Build System Details

### Vite Configuration (vite.config.ts)
The build uses a custom `entryConfig` function that:
1. Generates ESM and CJS formats via Vite's library mode
2. Uses `vite-plugin-dts` to generate rolled-up type definitions
3. Post-processes output with `minifyAndUMDPlugin` to create UMD bundles via esbuild
4. Duplicates `.d.ts` files as `.d.cts` for CommonJS compatibility (required by publint)

**Critical:** The `afterBuild` hook in the dts plugin copies `.d.ts` → `.d.cts` to satisfy dual ESM/CJS exports. Do not remove this.

### Package Exports
The `package.json` exports field provides dual ESM/CJS entry points with corresponding type definitions for each format. This ensures compatibility with both modern ESM projects and legacy CommonJS consumers.

## Publishing Workflow

Publishing is automated via GitHub Actions (`.github/workflows/publish.yml`):
1. Update `version` in `package.json`
2. Create and push a git tag matching `v*` pattern (e.g., `v0.1.4`)
3. GitHub Actions automatically builds and publishes to npm with `@whenessel` scope

**Never commit built files.** The `dist/` directory is generated during CI and should remain gitignored.

## Code Style Conventions

- Use `export function` for all patch functions (not arrow functions)
- Window parameter is always typed as `Window & typeof globalThis`
- All mocks should be side-effect free and idempotent (safe to call multiple times)
- Use `Object.defineProperty` with explicit descriptors rather than direct assignment
- Include JSDoc comments explaining usage context, especially cross-references to where functions are called

## Testing Context

This library is designed for consumption in test environments. When modifying mock behavior:
- Assume consumer tests depend on exact return values (especially geometry dimensions)
- Breaking changes require major version bumps
- Consider adding new optional parameters rather than changing existing behavior

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whenessel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
