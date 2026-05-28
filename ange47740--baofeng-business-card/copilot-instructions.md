## baofeng-business-card

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start Vite dev server with hot reload
npm run build        # Type-check + build for production
npm run build-only   # Build without type-checking
npm run preview      # Preview production build locally
npm run type-check   # Run TypeScript type checking (vue-tsc)
npm run lint         # Run all linters (oxlint + eslint, both with auto-fix)
npm run format       # Format code with Prettier
```

Node version requirement: `^20.19.0` or `>=22.12.0`

## Tech Stack

- **Vue 3** with Composition API (`<script setup>` style)
- **TypeScript** (strict mode) via `vue-tsc`
- **Vite** as build tool
- **Dual linting**: Oxlint (fast, auto-fix) + ESLint (rules via `eslint.config.ts`)
- **Prettier**: no semicolons, single quotes, 100-char line width (see `.prettierrc.json`)

## Architecture

This is a **Vue 3 SPA** (electronic business card / 電子名片) for 寶豐. Currently minimal:

- `src/main.ts` — app entry point, mounts to `#app`
- `src/App.vue` — root component
- No router or state management configured yet

### Path Alias

`@/` maps to `src/` (configured in both `vite.config.ts` and `tsconfig.app.json`).

### TypeScript Config Split

- `tsconfig.json` — references the two sub-configs
- `tsconfig.app.json` — app source files (strict, paths alias)
- `tsconfig.node.json` — build tool files (vite.config.ts, etc.)

---
> Source: [ange47740/baofeng-business-card](https://github.com/ange47740/baofeng-business-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
