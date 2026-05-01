## modular-refactoring-patterns

> Patterns for refactoring large monolithic files into modular structures


# Modular Refactoring Patterns

This rulecard is intentionally short. For step-by-step patterns and larger examples, see `docs/reference/modular-refactoring-patterns.md`.

## Default approach

- Extract **pure helpers** first (no I/O, no global state)
- Extract **types** next (reduce import cycles)
- Keep **public API** stable while moving internals

## Related rules

- `.cursor/rules/resolving-cyclic-dependencies.mdc`
- `.cursor/rules/no-barrel-files.mdc`

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
