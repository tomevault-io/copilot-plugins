## providers

> Adding or changing a provider backend (build / storage / credentials / submit / compute).


Adding a backend = implement one of the five interfaces from `src/core/types/providers.ts` as a named object and register it in `src/providers/index.ts`, which wires into `src/core/services/registry.ts`.

- The pipeline resolves a provider by its `name` (the value users put in `launch.config.ts`), so you **never** edit `src/core/build/pipeline.ts` to add a backend.
- Lazy-load heavy / optional SDKs (AWS, the native keyring) through `requireOptional` in `src/core/services/optionalDep.ts`, so a missing package becomes an actionable install hint instead of a stack trace.

---
> Source: [YosefHayim/launch-store](https://github.com/YosefHayim/launch-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
