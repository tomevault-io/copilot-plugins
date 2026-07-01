## capsummarize

> - **Build**: `bun run build` - Full production build

# AGENTS.md

## Build & Development Commands

- **Build**: `bun run build` - Full production build
- **Dev**: `bun run dev` - Development mode with watch
- **Type check**: `bun run type-check` (runs `tsc --noEmit`)
- **Lint**: `bun run lint` (ESLint on src/)
- **Format**: `bun run format` (Prettier)
- No test framework configured

## Code Style

- **TypeScript**: Strict mode enabled with `noImplicitReturns`, `noUncheckedIndexedAccess`
- **Formatting**: Prettier - single quotes, 2-space indent, 100 char width, trailing commas (es5)
- **Imports**: Use `.js` extension for local imports (ES modules); group external first, then local
- **Naming**: camelCase for variables/functions, PascalCase for types/interfaces/classes, UPPER_SNAKE_CASE for constants
- **Types**: Prefer explicit interfaces with JSDoc comments; avoid `any` (warn); prefix unused params with `_`
- **Error handling**: Use try/catch with `logger.error()` for errors; always handle Promise rejections with `.catch()`
- **Logging**: Use `logger` from `./utils/logger.js` (log, info, warn, error, debug); prefix logs with context `[Component]`

## Architecture

Chrome extension with: background service worker, content scripts, side panel UI. Source in `src/`, output in `dist/`.

---
> Source: [samuxbuilds/capsummarize](https://github.com/samuxbuilds/capsummarize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
