## curso-react

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a monorepo-style React learning project containing three separate Vite + React applications. Each project lives in its own directory with independent dependencies.

## Projects

| Directory | Stack | React Version |
|-----------|-------|---------------|
| `gentlemanreact/` | TypeScript + Vite + SWC | 19.0.0 |
| `03-counter-app/` | JavaScript + Vite + Babel | 19.1.0 |
| `04-first-steps/` | TypeScript + Vite + SWC | 19.2.0 |

## Development Commands

All projects share the same npm scripts. Run these from within each project directory:

```bash
# Start development server with HMR
npm run dev

# Production build (includes tsc for TypeScript projects)
npm run build

# Run ESLint
npm run lint

# Preview production build
npm run preview
```

**Package Manager:** Projects use Bun (bun.lock present) with npm fallback.

## Architecture Notes

- **Bundler:** Vite with either SWC or Babel for Fast Refresh
- **TypeScript:** Strict mode enabled with bundler module resolution
- **ESLint:** Flat config format (ESLint 9+) with typescript-eslint
- **No testing framework** is currently configured
- **No routing or state management** beyond React hooks

## TypeScript Configuration

TypeScript projects use a split configuration:
- `tsconfig.json` - References app and node configs
- `tsconfig.app.json` - Application source code settings
- `tsconfig.node.json` - Vite config compilation

---
> Source: [macor003/curso-react](https://github.com/macor003/curso-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
