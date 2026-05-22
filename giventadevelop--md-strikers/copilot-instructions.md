## squad-collage-split

> description: How to split MD Strikers squad collage images into per-member JPEGs via Sharp script

---
description: How to split MD Strikers squad collage images into per-member JPEGs via Sharp script
globs: scripts/split-squad-collage.mjs, public/images/md_strikers_media/squad/**/*
alwaysApply: false
---

# Squad Collage → Individual Member Images

## When to use

- A **single source image** contains a **uniform grid** of squad headshots (this project: **4×4**, **15** filled cells, **1 empty**).
- Goal: export per-cell JPEGs for the Squad UI or `public/` assets—**no AI splitter required** for a regular grid.

## Canonical paths

| Role | Path |
|------|------|
| **Inputs** | `public/images/md_strikers_media/squad/squad-N.jpg` (e.g. `squad-1.jpg`, `squad-2.jpg`) |
| **Output dir** | `public/images/md_strikers_media/squad/members/` |
| **Naming** | `squad-1` → `member-01.jpg` … `member-15.jpg`; `squad-N` (N≥2) → `member-N-01.jpg` … `member-N-15.jpg` |
| **Script** | [`scripts/split-squad-collage.mjs`](mdc:scripts/split-squad-collage.mjs) |

## Grid layouts (critical)

- **`squad-1.jpg`** (portrait sheet): **4×4** with one empty bottom-right cell → **15** crops.
- **`squad-2.jpg` / `squad-3.jpg`** (landscape contact sheet): **5×3** → 15 crops (`member-2-*`, `member-3-*`).
- **`squad-4.jpg`**: **3×1** (three portraits in one row) → **only** `member-4-01` … `member-4-03`. Older `member-4-04` … `15` are removed when you re-run split for squad 4.

Auto: **4×4** = `squad-1.jpg`; **5×3** = `squad-2`, `squad-3`; **3×1** = `squad-4`. Override: `npm run split-squad -- --grid 5x3 1`.

## Run

**Squad 1 only** (legacy names `member-01` … `member-15`):

```bash
npm run split-squad
```

**Squad 2 and 3** (outputs `member-2-NN` and `member-3-NN`):

```bash
npm run split-squad-2-3
```

**Squad 4** (`member-4-01.jpg` … `member-4-15.jpg`):

```bash
npm run split-squad-4
```

**Any squad numbers** (space-separated):

```bash
npm run split-squad -- 2 3
node scripts/split-squad-collage.mjs 1 2 3
```

**Dependency:** `sharp` (already in `package.json`).

## Grid logic (do not guess in chat—read the script)

- **4 columns × 4 rows** → 16 cells in **row-major** order (left→right, top→bottom).
- **Skip index 15** (`SKIP_INDEX = 15`): **bottom-right** cell is treated as **empty** and not exported.
- **Cell bounds:** `left/top/width/height` from **rounded** fractional splits so the full image width/height is covered without gaps (`cellBounds` in script).

## If the layout changes

1. **Empty cell elsewhere:** change `SKIP_INDEX` (0–15) to match **row-major** index of the empty slot.
2. **Different grid (e.g. 3×5):** update `COLS`, `ROWS`, loop bounds, and skip logic in `scripts/split-squad-collage.mjs`.
3. **Visible white gutters / half-cutoff faces in crops:** the script applies **outer margin** (`outerPct`) on the full sheet and **per-cell inset** (`cellInsetRatio`) so cuts stay inside each portrait. Defaults are in `LAYOUTS` in `scripts/split-squad-collage.mjs`. Tune per run, e.g. `npm run split-squad -- --outer-pct 0.03 --cell-inset-ratio 0.045 4`.

## Anti-patterns

- **❌** One-shot generative image tools to “split” a grid—use this **Sharp** script instead.
- **❌** Hardcoding pixel cell sizes—always derive from **source image dimensions** unless the asset is fixed forever.

## Related

- [`package.json`](mdc:package.json) — `split-squad`, `split-squad-2-3`, `split-squad-4`.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
