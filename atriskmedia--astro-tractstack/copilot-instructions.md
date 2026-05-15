## tailwind-constraints

> TractStack Tailwind — project-local rule (astro-tractstack); not a global Cursor rule


# Tailwind constraints (MUST follow)

This file applies when working in the **astro-tractstack** repo (solo folder or multi-root workspace). It is **not** a user-global rule.

## Forbidden

- **Arbitrary values** in Tailwind brackets: do **not** use `min-h-[...]`, `w-[12rem]`, `max-w-[...]`, `h-[calc(...)]`, `top-[3px]`, or any `-[...]` utility. Use named scale utilities (`max-w-xs`, `min-h-96`, `max-h-screen`, etc.) or non-Tailwind inline styles only when a precise one-off value is unavoidable.
- **Font weights** other than bold for emphasis in UI copy: do **not** use `font-medium`, `font-semibold`, or `font-normal` for hierarchy where the project standard is **`font-bold`** (use `font-bold` for strong emphasis; default body text may remain unweighted).

## Breakpoints

- Use **only** these responsive prefixes: **`xs:`** (if present in config), **`md:`**, **`xl:`**.
- Do **not** introduce `sm:`, `lg:`, `2xl:`, or other breakpoints unless an existing file already does and you are matching local patterns.

## When in doubt

Prefer the default Tailwind spacing/sizing scale (`p-4`, `max-w-2xl`, `h-screen`, `min-w-0`, etc.) over inventing bracket utilities.

---
> Source: [AtRiskMedia/astro-tractstack](https://github.com/AtRiskMedia/astro-tractstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
