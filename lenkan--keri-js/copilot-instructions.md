## keri-js

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KERI-JS is a TypeScript implementation of KERI (Key Event Receipt Infrastructure), a cryptographic key management and identity framework. It is a single npm package with multiple export entry points.

### Source layout (`src/`)

- **`cesr/`** — Composable Event Streaming Representation: low-level encoding/decoding primitives
- **`core/`** — KERI core logic: key events, credentials, signing, verification
- **`controller/`** — Identifier controller
- **`encoding/`** — Base64 and UTF-8 encoding utilities
- **`storage/`** — Storage backends (e.g. SQLite)
- **`witness/`** — Witness node implementation

## Commands

```sh
npm run build           # Compile TypeScript to dist/
npm run test            # Run all unit tests (src/**/*.test.ts)
npm run check           # TypeScript type-check (no emit)
npm run lint            # Biome lint
npm run format          # Biome formatting (write)

npm run test:interop      # Interop tests (requires KERIpy demo running)
npm run test:vector       # Cross-impl test vectors (test_vectors/)
```

Tests use the native Node.js test runner. Unit test files live alongside source files (`src/**/*.test.ts`).

## TypeScript & Code Style

- `strict: true` enabled
- Build output goes to `dist/`
- Biome handles linting and formatting (`biome.json` at root)
- Cryptography uses `@noble/*` libraries exclusively

---
> Source: [lenkan/keri-js](https://github.com/lenkan/keri-js) — distributed by [TomeVault](https://tomevault.io/claim/lenkan).
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
