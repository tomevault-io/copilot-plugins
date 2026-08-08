## opencode-observability-plugin

> - Prefer dependent aliases over duplicated external shapes. Use utility types such as `Parameters`, `Awaited`, `NonNullable` etc for this.

# Agent Instructions

## TypeScript

- Prefer dependent aliases over duplicated external shapes. Use utility types such as `Parameters`, `Awaited`, `NonNullable` etc for this.
- Avoid hand-maintained partial copies of external API types when a stable plugin-exported type can express the relationship.
- Do not create named argument object types when the argument is used by only one function or method; keep that object type inline at the parameter.
- Only create named types for reusable state, exported domain shapes, or types that other types are intentionally derived from.
- Do not add explicit return types by default. Use inference unless the function is a critical boundary where the return type is part of the contract.
- When checking an object against an interface without forcing an explicit return annotation, prefer `satisfies`.
- Do not wrap single built-ins or one-line property math in helper functions; e.g. having a helper like `stringify` that just does `JSON.stringify` inside.

## Verification

- Run `pnpm run format` and `pnpm run build` after edits.

---
> Source: [langfuse/opencode-observability-plugin](https://github.com/langfuse/opencode-observability-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
