## tsdown-dist-layout-in-tests

> Use tsdown dist root paths in TS test mappings


# tsdown dist layout for test path mappings

When packages are built with `tsdown`, declaration outputs are emitted in the package `dist/` root as `.d.mts` files (for example `dist/types.d.mts`), not under `dist/exports/*.d.ts`.

## Rules

- In temporary `tsconfig` fixtures used by tests, map package paths to `dist/*.d.mts` or `dist/*`.
- Do not assume `dist/exports/` exists unless the package explicitly emits that structure.
- Prefer wildcard mappings (`dist/*`) for subpath imports that may include both `.mjs` and `.d.mts` outputs.

## Why

- Prevents module resolution failures after tsup -> tsdown migrations.
- Keeps integration tests aligned with the actual package export artifact layout.

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
