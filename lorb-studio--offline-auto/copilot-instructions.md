## offline-auto

> Your web app works offline. Three lines, zero config, auto request queue.

# offline-auto

Your web app works offline. Three lines, zero config, auto request queue.

## Commands

npm run build    # compile TypeScript
npm test         # run all tests (Vitest)
npm run typecheck # type-check without emit (tsc --noEmit)

## Stack

- TypeScript (ESM only)
- Vitest for testing
- Zero runtime dependencies

## Code Style

- ESM imports/exports only. No CommonJS
- Strict TypeScript (`strict: true`)
- No default exports — use named exports

## Boundaries

- Do not modify `.env` or credentials
- Do not run `npm publish` without maintainer approval
- Do not add runtime dependencies without discussion

---
> Source: [lorb-studio/offline-auto](https://github.com/lorb-studio/offline-auto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
