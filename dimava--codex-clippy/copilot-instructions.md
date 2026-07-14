## codex-clippy

> Repo notes for AI coding agents (Codex, Claude, others).

# AGENTS.md

Repo notes for AI coding agents (Codex, Claude, others).

## What this repo is

Microsoft Clippit/Clippy ACS extracted into:

- 43 animated WebPs (one per ACS animation), in `assets/animations/webp/`
- A Codex-compatible pet package in `assets/pet/` (matches the `/hatch` skill spec: 1536x1872 transparent WebP atlas with 192x208 cells, 8 cols x 9 rows, plus `pet.json`)

The Codex pet is published in releases as `clippy-pet.zip`. To install:

```powershell
# unpack pet.json + spritesheet.webp into:
mkdir -p $HOME/.codex/pets/clippy
# ...drop both files there
```

Then `/pets clippy` in Codex.

## Layout

```
third-party/                 source files (NOT licensed for redistribution)
  clippitMS/CLIPPIT.ACS      Microsoft Agent character file (the source of truth)
  clippitMS/CLIPPIT.ACG
  clippitMS/readme.txt
  decompress.wasm            ACS image decompressor from computernewb/MSAgent-Chat
assets/
  animations/                derived: per-animation webp + sheets + strips + manifest.json
  pet/                       derived: 8x9 atlas + pet.json + contact-sheet + review.json
scripts/                     all Bun TypeScript, use sharp for image work
  extract-acs-images.ts      ACS -> .out/acs-images/*.png + images.json
  split-clippy-animations.ts orchestrator: runs extract, then writes assets/animations/*
  build-clippy-pet.ts        writes assets/pet/* and installs to ~/.codex/pets/clippy/
  update-readme-gallery.ts   regenerates the README animation table from the manifest
.out/                        gitignored scratch/build dir
```

The animation manifest schema: each entry has `index` (1-based ACS animation ordinal), `name`, `frame_count`, `webp_file`, `sheet_file`, `strip_file`, durations, and per-frame `image_index` pointing into the ACS image table.

## Regeneration

```powershell
bun scripts/split-clippy-animations.ts   # invokes extract-acs-images.ts internally
bun scripts/build-clippy-pet.ts          # also installs into ~/.codex/pets/clippy/
bun scripts/update-readme-gallery.ts
```

Build is reproducible: re-running produces byte-identical outputs (modulo paths in `assets/pet/review.json`). Multiple ACS animations share the same underlying frame data, so some webp files are byte-identical by design (e.g. CONGRATULATE/GETWIZARDY).

## Conventions

- No Python. All scripts are Bun TypeScript (`#!/usr/bin/env bun`, top-level await, sharp for image work, raw RGBA buffers + sharp.composite for layout, `sharp([..pngBuffers], { join: { animated: true } })` for animated WebP).
- Filename convention: animation files are `NN-NAME.{webp,png}` where `NN` is the 1-based ACS animation ordinal. Do not introduce filenames based on cumulative "global frame" counters - that mapping is meaningless.
- Sheets do not include per-cell labels. Frame numbers within an animation are not meaningful identifiers; the only stable identifier per frame is the `image_index` in `manifest.json`.

## License caveats

`LICENSE.md` ("The Piracy License") grants permissive rights to original work only. The contents of `third-party/` and the derivative `assets/` are explicitly NOT licensed for redistribution by this repo. Treat any agent task that promotes wider distribution of those artifacts (e.g. mirroring to other repos, publishing to package registries) as out of scope unless the user has explicitly authorized it for that target.

## /hatch skill compatibility

`assets/pet/spritesheet.webp` and `assets/pet/pet.json` together form a Codex pet that satisfies the `hatch-pet` skill's acceptance criteria: 1536x1872 transparent atlas, 192x208 cells, the 9 standard rows (`idle`, `running-right`, `running-left`, `waving`, `jumping`, `failed`, `waiting`, `running`, `review`).

This pet is built by extraction + sampling, not by `$imagegen`. That bypasses the normal `/hatch` flow but produces a compatible artifact; do not run `/hatch` over this repo.

## Build a custom pet from these animations

The Clippy pet picks one ACS animation per row and samples evenly through its frames. To build a different pet from the same source — e.g. a "writing-focused Clippy" or a remix that uses different animations — edit `ROW_SPECS` in `scripts/build-clippy-pet.ts`:

```ts
const ROW_SPECS: RowSpec[] = [
  // [state, row index, frame count, ACS animation name]
  ["idle",          0, 6, "IDLE1_1"],
  ["running-right", 1, 8, "GESTURERIGHT"],
  ["running-left",  2, 8, "GESTURELEFT"],
  ["waving",        3, 4, "WAVE"],
  ["jumping",       4, 5, "GETATTENTION"],
  ["failed",        5, 8, "ALERT"],
  ["waiting",       6, 6, "IDLESIDETOSIDE"],
  ["running",       7, 6, "PROCESSING"],
  ["review",        8, 6, "CHECKINGSOMETHING"],
];
```

Rules for editing:

- Row state names, row order, frame counts per row, atlas geometry, cell geometry, and any other shape requirements are owned by the `/hatch` skill. Treat the `/hatch` skill (`${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet/SKILL.md`) as the authoritative spec and update the constants at the top of `scripts/build-clippy-pet.ts` (`CELL_W`, `CELL_H`, `COLUMNS`, `ROWS`, the inner-box and offset values inside `fitToCell`, and the `ROW_SPECS` row count/order) if that spec changes. Do not hardcode "rules" here that duplicate the skill.
- `ACS animation name` must be one of the 43 names in `assets/animations/manifest.json` (also visible as filenames in `assets/animations/webp/`, e.g. `01-GESTURELEFT`, `41-IDLEATOM`).
- The script samples frames evenly through each source animation via `sampleIndices`, so a long source animation becomes a smaller number of evenly-spaced poses.
- After editing, re-run `bun scripts/build-clippy-pet.ts`. It writes `assets/pet/spritesheet.webp` + `pet.json` and installs them at `~/.codex/pets/clippy/`. To install under a different name, change the `id` in the generated `pet.json` (or edit it in the script before re-running) and move the folder to `~/.codex/pets/<your-name>/`.

The pipeline is: read the animations manifest -> for each row, pick frames from one source animation -> trim/scale/center each frame into a cell -> composite into the atlas -> write PNG, lossless WebP, and a labeled contact sheet for QA. No `$imagegen` involved.

---
> Source: [Dimava/codex-clippy](https://github.com/Dimava/codex-clippy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
