## trueforge

> - `ISessionStore` is the session/turn persistence contract (no streaming/SSE); Postgres, SQLite, and `InMemorySessionStore` MUST implement it, and shared behavior MUST live in `storeContractSuite.ts` (backend wrappers only bind a store).

- `ISessionStore` is the session/turn persistence contract (no streaming/SSE); Postgres, SQLite, and `InMemorySessionStore` MUST implement it, and shared behavior MUST live in `storeContractSuite.ts` (backend wrappers only bind a store).
- Changing an `ISessionStore` method, input, error, or semantic MUST update the interface, every implementation, and the contract suite in the same change.
- Path filters for the `store` filter in `.github/workflows/ci.yml` MUST stay synchronized when store or contract-test paths are added, moved, or renamed.

---
> Source: [truefoundry/trueforge](https://github.com/truefoundry/trueforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
