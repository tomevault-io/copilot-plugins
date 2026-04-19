## numismatist

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands
- Build: `npm run build` 
- Lint: `npm run lint`
- Test all: `npm test`
- Test single file: `npm test -- path/to/test.js`
- TypeCheck: `npm run typecheck`

## Code Style Guidelines
- Use ESLint with Prettier for formatting
- Prefer TypeScript with explicit type annotations
- Follow naming conventions:
  - camelCase for variables and functions
  - PascalCase for classes and React components
  - UPPER_CASE for constants
- Error handling: Use try/catch blocks and custom error classes
- Imports: Group imports by external libs, then internal modules
- Components: One component per file
- Comments: Only add when necessary to explain complex logic
- Test coverage: Aim for 80%+ coverage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qfennessy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
