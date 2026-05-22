## csharp-analyzers

> description: Enforce analyzer compliance and hook validation in Rust plugins

---
description: Enforce analyzer compliance and hook validation in Rust plugins
globs: ["*.cs"]
alwaysApply: true
---

# C# Analyzer Compliance

- All code MUST follow Microsoft's official C# coding standards.
- You MUST fix all diagnostics from:
  - `dotnet build`
  - IDE analyzers (CAxxxx, CSxxxx, IDExxxx)
- Suppressions like `#pragma warning disable` are forbidden.

❌ Never allowed:
```csharp
#pragma warning disable CS8618
#pragma warning disable IDE0051
```

Fix issues properly instead — rewrite logic, initialize values, or redesign features.

---

# Rust Hook Validation

- All hooks (`OnPlayerInit`, `OnEntityDeath`, etc.) MUST match exact Rust uMod API names (case-sensitive).
- ALWAYS validate via:
  - `search_codebase("OnHookName")`
  - `search_web("uMod OnHookName Rust")`
- NEVER invent or guess hook names.
  - ❌ `OnPlayerInit()` — invalid
- Invalid hooks silently fail and break plugin logic.

> Fix analyzers. Validate hooks. No suppressions. No guesses.

---
> Source: [publicrust/rust-template](https://github.com/publicrust/rust-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
