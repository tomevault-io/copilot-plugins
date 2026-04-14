## the-bridge-hack

> TypeScript conventions for this project


# TypeScript Conventions

- Avoid `any` (use `unknown` + narrowing).
- Prefer explicit types at module boundaries (public exports, API inputs/outputs).
- Don’t use type assertions (`as X`) unless there’s no better modeling option.
- Favor small, composable utilities over mega “utils” files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pillin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Pillin)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
