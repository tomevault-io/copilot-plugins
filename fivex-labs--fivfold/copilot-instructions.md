## fivfold-architecture

> FivFold architecture rules for core, ui, api


# FivFold CLI Architecture

When modifying core, ui, or api packages, follow [AGENTS.md](../../AGENTS.md).

- **Existing files:** Use ts-morph AST, never regex/string replace
- **New files:** Handlebars templates
- **All writes:** Via VFS; stage then commit atomically
- **Manifests:** Define Kits in manifests/*.kit.json; no hardcoded stack permutations

---
> Source: [Fivex-Labs/fivfold](https://github.com/Fivex-Labs/fivfold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
