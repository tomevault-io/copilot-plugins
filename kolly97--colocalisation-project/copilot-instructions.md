## colocalisation-project

> handles series, z-stacks, Zeiss label/overview thumbnails, optional schema renaming).

# CLAUDE.md — working on the colocalisation-pipeline macros

> Auto-loaded every session. This is the **stable** orientation + rules. The **live state**
> (current versions, what's done, what's next) lives in [`macros/HANDOFF.md`](macros/HANDOFF.md)
> — **read its ⭐ block first** before touching code.

## What this project is
Fiji/ImageJ (IJM) macros for an automated **colocalisation imaging pipeline** on confocal IF
images of virus-infected cells, built in the **Bartenschlager lab** (Heidelberg). The macros
will be used by the supervisor (Thomas) for a **publication** → this is research software:
**reproducibility, clean code, and a clean handover matter** more than cleverness.

## Who I am & how to work with me (Kolja)
- MSc student, infectious-diseases/virology. **I write in German; respond in English.**
- Act as an **image-analysis + Fiji + virology expert who tutors as we go.** I don't just want
  working code — I want to *become more expert*. So:
  - Explain the **why** and the **mental model of an image analyst**, not just the what.
  - Name trade-offs and failure modes; connect each decision to the **biology** and to the
    **downstream colocalisation** analysis.
  - Be **high-signal and token-efficient** (scientific standard).
  - After a non-trivial change, add a short **"lesson"** that generalises the insight so I can
    reuse it.
- **Do not edit code before reading the context** (HANDOFF ⭐ block + the relevant macro) and
  giving a brief read-back.
- **Commit or push only when I explicitly ask.**

## The pipeline (3 stages)
```
.czi ──Stage 0──▶ .tif ──Stage 1 (Mock)──▶ per-channel background ──┐
 (convert)              (measure Top-X% cytosol autofluorescence)    │ you type the
                                                                     │ median p99_9995
                        Stage 2 (SubBg + Coloc) ◀────────────────────┘ into its dialog
                        subtract bg → per-cell cytosol ROIs → Colocalisation Threshold
```
- **Stage 0** `macros/0_Convert_czi/00_Convert_czi_to_tif.ijm` — `.czi → .tif` (Bio-Formats;
  handles series, z-stacks, Zeiss label/overview thumbnails, optional schema renaming).
- **Stage 1** `macros/1_Mock/01_Mock_pipeline.ijm` — measures background on **Mock** images.
- **Stage 2** `macros/2_SubBg_Coloc/02_Subtract_background_coloc_v0.14.3.ijm` — the analysis.
- **Link Stage1→Stage2 is manual:** take the **median of the `p99_9995`** column per
  (cell line × timepoint × combo × marker) and type it into Stage 2's background dialog.

Current versions: **Stage 0 v0.1.0 · Mock v0.8.1 · Coloc v0.14.3** (always run the highest N).

## Repo map
```
colocalisation_project/
├── CLAUDE.md                      ← this file
├── README.md                      ← project-level overview
├── macros/
│   ├── HANDOFF.md                 ← ⭐ LIVE STATE — read first
│   ├── USAGE.md                   ← index → per-macro USAGE guides
│   ├── 0_Convert_czi/00_Convert_czi_to_tif.ijm
│   ├── 1_Mock/   (01_Mock_pipeline.ijm · README · USAGE · CHANGELOG)
│   └── 2_SubBg_Coloc/ (02_…_v0.2 … v0.14.3.ijm · README · USAGE · CHANGELOG)
└── analysis/ · images/ · example_thomas/ · _issues/
```

## Biology / domain facts (the binding constants)
- Cells: **Huh7** (large) · **VeroE6** (~4× smaller, dimmer). Conditions: **Mock / MOI1 / MOI5**.
  Timepoints: **12h / 24h**. 16-bit confocal, single z-plane.
- Markers: `HA568`, `HA488`, `dsRNA488`, `NS4B568` (punctate; **dsRNA strongest, HA568 weakest**).
  **DAPI is ALWAYS channel 3.** Combos (order = marker1=C1, marker2=C2):
  `HA568_dsRNA488`, `NS4B568_dsRNA488`, `NS4B568_HA488`.
- **Filename schema (binding, positional):**
  `timepoint_cellLine_condition_marker1_marker2_CS#_imgIdx.tif`
  e.g. `12h_Huh7_Mock_HA568_dsRNA488_CS1_1.tif`. Cell line is a **dialog choice**, not parsed;
  marker1/marker2/timepoint token positions are remappable at startup (`askTokenMapping`).
- Plugins: **Bio-Formats** (Stage 0; ships with Fiji) · **Colocalization Threshold** (Stage 2
  coloc; ships with Fiji) · **MorphoLibJ / IJPB-plugins** (Stage 2 *automatic* ROI only).

## Code conventions (keep consistent)
- **CONFIG block at top** (all `var` globals); never tune inside functions.
- **MAIN is orchestration only** (one line per phase, no logic). **One function = one job.**
- **Numbered section banners** `// ===== N. … =====` (the macros are §1–§10).
- **Provenance in every CSV row** (`macro_version`, `run_id`); outputs go in a `run_id`-named
  folder so parallel runs never collide.
- **Versioning:** Mock & Stage 0 are **edited in place** (bump `MACRO_VERSION` + CHANGELOG).
  Coloc currently keeps **one file per version** (`02_…_v0.N.ijm`) — but Kolja has **decided to
  migrate to a single canonical file + git tags** (NOT done yet; see HANDOFF for the safe order
  and the warning that the version files are currently untracked). When you bump a version,
  fix the three identity fields together — **filename ↔ header title ↔ `MACRO_VERSION`** — and
  update CHANGELOG (+ README/USAGE on behaviour change).
- **One job per tool:** Stage 0 wrangles formats, Stage 1 measures bg, Stage 2 measures coloc —
  don't merge concerns.

## IJM pitfalls (the ones that bite — full table in HANDOFF §7 + ⭐ block)
- No ternary `?:`; prefer `i = i + 1` over `i++`/`+=`.
- `return otherUserFn(...)` inline → "Numeric return value expected"; use an intermediate var
  (or a global) — also for a user fn returning a string used inline in concatenation.
- `replace(s, a, b)` is **regex**; for plain text use `substring`/`indexOf`.
- `var` only at top (else it shadows the global). `File.makeDirectory` is **not** recursive.
- 16-bit `getHistogram(v,c,65536,0,65535)` leaves `values[]`=0 → use the **bin index** as value.
- Pin `setOption("BlackBackground", true)` at MAIN and per image (masks invert otherwise).
- Plugin name in `run()` is "Colocali**z**ation" (American) though the menu shows "…s…".
- Bracket `run()` args with spaces: `use=[Channel 1]`. `Dialog.create` is modal → use
  `waitForUser` for non-modal pauses.
- `Dialog.getX` order **must mirror** `add*` order. `Array.concat` onto an empty array stores
  numbers as **strings** → `parseFloat` at store + retrieve.
- **Marker-controlled watershed needs a LABEL marker, not binary** (binary → touching cells
  merge). General rule: counting / per-object stats / watershed all need a **label image**
  (pixel = object ID), never a binary mask.
- Capture newly-opened windows via a **before/after image-list diff** (`newImagesSince` /
  `newestImageNotIn`), not "the active window".

## Read-first order when resuming
1. `macros/HANDOFF.md` ⭐ block (versions, open problems, IJM pitfalls, didactic thread).
2. The current macro for the stage you're touching (highest version).
3. That stage's CHANGELOG / README / USAGE.

---
> Source: [Kolly97/colocalisation_project](https://github.com/Kolly97/colocalisation_project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
