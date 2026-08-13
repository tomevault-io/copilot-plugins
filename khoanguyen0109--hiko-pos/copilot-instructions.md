## typescript-no-explicit-any

> TypeScript in pos-backend must not use the any type


# TypeScript: no `any`

In `pos-backend/`, do **not** use TypeScript’s `any` type.

- Use `unknown` for values that must be narrowed (e.g. `catch` clauses, parsed JSON).
- Use concrete interfaces or `import type` from `types/` (e.g. `MongoFilter`, model payloads).
- For dynamic MongoDB filters, use `MongoFilter` (`Record<string, unknown>`) or Mongoose `FilterQuery<T>` when the document type is known.
- ESLint rule: `@typescript-eslint/no-explicit-any` is enabled; `npm run lint` must pass.

---
> Source: [Khoanguyen0109/Hiko-POS](https://github.com/Khoanguyen0109/Hiko-POS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
