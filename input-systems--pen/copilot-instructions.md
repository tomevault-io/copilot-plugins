## pen-core-engineering

> Core engineering rules for consistent Pen OSS contributions


# Core Engineering Rules

- Keep Pen headless and extension-first: implement behavior/state, not opinionated styling.
- Preserve the architecture split: schema/data, headless behavior, rendering UI.
- Respect package boundaries:
  - `@pen/types` for contracts and lightweight helpers.
  - `@pen/core` for runtime editor behavior.
- Code against stable interfaces/adapters; avoid coupling shared code to concrete framework or CRDT internals.
- Use `editor.apply(ops, { origin })` as the canonical mutation path, and do not bypass the pipeline for document writes.
- Set operation origin intentionally (`user`, `ai`, `collaborator`, `input-rule`, etc.) so undo/hooks/diagnostics stay correct.
- Keep `onBeforeApply` transforms bounded and deterministic; use them for auth/suggest/input-rule transformations only.
- Prefer non-fatal behavior in runtime paths: drop invalid mutations with diagnostics rather than crashing editor sessions.
- Keep normalization incremental and idempotent (dirty-block focused; repeated passes should not introduce new changes).
- For document-wide features (export/search/history), traverse all blocks including nested/layout children.
- Prefer consistency with existing local patterns over introducing new abstractions.
- Keep public API and package exports stable unless the task explicitly requires API change.

---
> Source: [input-systems/pen](https://github.com/input-systems/pen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
