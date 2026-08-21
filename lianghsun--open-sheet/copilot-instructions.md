## open-sheet

> This file describes the **framework repo**. If you are authoring workbooks in a scaffolded workspace, you want the `sheet-authoring` skill instead.

# open-sheet — repository guide

This file describes the **framework repo**. If you are authoring workbooks in a scaffolded workspace, you want the `sheet-authoring` skill instead.

## Repo layout

pnpm + Turbo monorepo.

| Path | Package | Purpose |
| --- | --- | --- |
| `packages/core` | `@open-sheet/core` | Compiler, placement, references, formula engine, style bridge, export, viewer, Vite plugin, `open-sheet` CLI |
| `packages/cli` | `@open-sheet/cli` | Scaffolder and project template |
| `packages/mcp` | `@open-sheet/mcp` | MCP server over Streamable HTTP |
| `apps/demo` | private | Local framework consumer, dogfood target |

Shared config at the root: `biome.json`, `turbo.json`, `pnpm-workspace.yaml`, `tsconfig.base.json`.

## Commands

```bash
pnpm dev          # run the demo against local core
pnpm build        # build all packages
pnpm typecheck    # tsc across the monorepo
pnpm check        # biome: format, lint, organize imports
pnpm check:fix    # auto-fix
pnpm test         # vitest
```

Filter to one package: `pnpm core <script>`, `pnpm cli <script>`, `pnpm mcp <script>`.

## The one invariant

**No A1 address is ever authored by hand — not in `apps/demo`, not in fixtures, not in tests that represent user code.**

The whole premise of open-sheet is that the framework owns every coordinate. A test fixture containing `=SUM(B2:B13)` written by a human is a fixture that stops testing the thing that matters. Tests may *assert* on A1 output; they must not *author* it.

The one exception is `raw()`, the deliberate escape hatch. It is expected to show `#NOT_EVALUATED` in the viewer.

## Core pipeline

Understand this before touching `packages/core`:

```
sheets/<id>/index.tsx
  → (1) evaluate JSX via our own jsx-runtime (no react-dom, no DOM at all)
  → Block tree
  → (2) measure   each block reports its intrinsic { rows, cols }
  → (3) place     assign every block an origin; collisions are a bug
  → Grid model + anchor registry
  → (4) resolve   Refs → concrete A1 / defined names
  → (5) evaluate  topological evaluation → the numbers the viewer shows
  → emit: viewer | .xlsx | .csv | .html | .pdf
```

Sizes are discrete (rows × cols), so measurement is pure arithmetic. This is the key divergence from open-doc, which must measure real DOM to paginate. Keep it that way: **the compiler must never require a browser.** Export runs in Node.

## Conventions

- **Licenses matter.** `@open-sheet/core` ships MIT. Never add a GPL/AGPL dependency — this is why we do not use HyperFormula and wrote our own expression AST. Check the license before adding anything to `core`.
- **Dependencies in `core` are load-bearing.** It ships to users; every dependency is install size for them.
- **Never invent a number.** If the evaluator cannot compute a cell, it renders `#NOT_EVALUATED`. A plausible-looking wrong number in a financial model is the worst failure this project can have.
- **Biome must pass before commit.** Run `pnpm check`.
- **Code style:** no comments unless the *why* is non-obvious. Well-named code needs no narration; hidden constraints and subtle invariants do.

## Naming

`sheets/<id>/index.tsx` · `open-sheet.config.ts` · `themes/<id>.md` + `<id>.demo.tsx` · CSS vars `--os-*` · comment marker `@sheet-comment` · current-position file `node_modules/.open-sheet/current.json` · dev port **5373**

## Cached formula results — a decision worth knowing

The xlsx writer stores a cached result next to each formula, so viewers that
cannot compute (Preview, GitHub, most mobile apps) still show numbers. It also
sets `fullCalcOnLoad`.

Two things were measured, not assumed:

- **Excel** honours `fullCalcOnLoad` and recalculates on open.
- **LibreOffice** does not. Its default for xlsx is *never recalculate on load*,
  so it displays our cached results until the user edits a cell.

That is why the CI recalculation check exports with `cacheValues: false`. With
cached results present, LibreOffice reads back the numbers we wrote and the
check passes even if `serialize()` emits nonsense — it would be a test of
nothing. Do not "simplify" it by dropping that flag.

Caching values is only safe *because* that check proves our evaluator and a real
spreadsheet engine agree about what our formulas mean.

---
> Source: [lianghsun/open-sheet](https://github.com/lianghsun/open-sheet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
