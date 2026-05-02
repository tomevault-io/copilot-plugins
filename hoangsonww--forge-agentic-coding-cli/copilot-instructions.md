## typescript

> TypeScript style rules, scoped to .ts files


# TypeScript rules

- Strict mode is on. Never use `any` in production code without a
  comment explaining why. The one accepted exception is
  `src/tools/registry.ts`.
- Prefer `readonly` and immutable data flow. Prefer function modules
  over classes, except in `src/models/*` where the provider-class
  shape is established.
- Return `Result<T, E>`-shaped objects for expected failures. Throw
  only for programmer errors.
- Throw `ForgeRuntimeError` from `src/types/errors.ts`, never bare
  `Error`.
- Comments explain **why**, not what. Delete comments that only
  narrate the code. Never leave a comment describing the change you
  just made.

---
> Source: [hoangsonww/Forge-Agentic-Coding-CLI](https://github.com/hoangsonww/Forge-Agentic-Coding-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
