## strong-typing

> Enforce strict typing and type-safe development practices


# Strong Typing Standards

- Prefer explicit domain types and value objects over primitive obsession.
- Do not use `any` unless there is a documented and temporary migration exception.
- Model external input as unknown/untrusted and validate before use.
- Represent optionality and failure explicitly with union types or result objects.
- Keep function signatures precise: typed parameters, typed returns, and typed errors where applicable.
- Centralize shared contracts/types and avoid ad-hoc duplicate type declarations.
- Favor immutable data structures and readonly types for safety.

---
> Source: [fidelrojas-Assure/TaskMaster-Pro](https://github.com/fidelrojas-Assure/TaskMaster-Pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
