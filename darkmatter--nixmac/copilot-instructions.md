## native-state-package

> Where TypeScript state and data-fetching code lives


# State and data fetching — `packages/state`

TypeScript code for **state, data fetching, storage, and cache** belongs in **`packages/state`** (import: `@nixmac/state`).

- Query hooks, selectors, and client-side projections live here — not scattered in `apps/native/src/components/`.
- Server/async data from Rust: use oRPC + React Query (see `native-orpc` rule); wire hooks in `packages/state` when shared across views.
- Existing Zustand slices (`viewmodel`, `ui`, `onboarding`) hold **client-side** and **event-projected** state only — do not extend them for new IPC fetch paths.

---
> Source: [darkmatter/nixmac](https://github.com/darkmatter/nixmac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
