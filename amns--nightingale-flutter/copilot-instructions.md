## nightingale-flutter

> Port the Nightingale music notation engine from legacy C/Carbon/QuickDraw to a modern

# Nightingale Modernization — Claude Code Project Guide

## Mission

Port the Nightingale music notation engine from legacy C/Carbon/QuickDraw to a modern
Rust core with a Flutter UI, preserving the engraving algorithms that are the project's
real value. The owner (Geoff Chirgwin) is ruthless about cutting cruft — when in doubt,
delete it.

## Repository

- Upstream (OG C source): https://github.com/AMNS/Nightingale (branch: `develop`)
- This repo (Rust/Flutter port): https://github.com/AMNS/nightingale-flutter (branch: `main`)
- License: MPL-2.0

## Codebase Stats (develop branch)

- 195K lines across 277 source files (C, with `.cp` extension on old Mac)
- Core files in `CFilesBoth/`, `CFilesEditor/`, `Utilities/`, `Precomps/`
- ~650 QuickDraw drawing calls
- ~1,118 Carbon dialog API calls
- ~309 Carbon control API calls  
- ~414 Carbon window API calls

## Source Directory Map

```
src/
├── CFilesBoth/       68K lines — CORE: shared code (data model, drawing, file I/O,
│                      beams, layout, engraving). THIS IS WHAT WE'RE PORTING.
├── CFilesEditor/     76K lines — Editor operations (editing, MIDI I/O, dialogs,
│                      insert/delete, undo, reformatting). PARTIALLY PORT.
├── Utilities/        28K lines — Utility functions. PORT SELECTIVELY.
├── Precomps/          8K lines — Header files / type definitions. PORT FIRST.
├── FilesSearch/       5K lines — Score search. PORT LATER.
├── MIDI/              3K lines — MIDI (CoreMIDI already). PORT.
├── Headers/           3K lines — App-level headers. REFERENCE ONLY.
├── CFilesCarbon/      92 lines — Carbon stubs. DELETE.
├── CarbonPrinting/   1.3K lines — Carbon printing. DELETE (use PDF).
├── CarbonNavServices/ 735 lines — Carbon file dialogs. DELETE.
├── CarbonTEField/     728 lines — Carbon text fields. DELETE.
├── CustomLDEF/        165 lines — Custom list definition. DELETE.
├── CustomMDEF/       1.2K lines — Custom menu definition. DELETE.
```

Other directories:
```
doc/                   Tech notes, format docs (NgaleFileFormatStatus.txt is key)
MiscellaneousCode/     Experimental/unused code (NTypesNEW.h has proposed N106 types)
Resources/             Icons, bitmaps, .rsrc files
English.lproj/         Localization
```

## Architecture Decisions (FINAL — do not re-debate)

1. **Rust core** — all data model, engraving, layout, file I/O
2. **Flutter UI** via `flutter_rust_bridge` — Rust is authoritative, Flutter just renders
3. **No Carbon/QuickDraw** — zero legacy Mac APIs in the new code
4. **MusicXML import/export** — first-class citizen (interop with Dorico, MuseScore, etc.)
5. **SMuFL fonts** — replace Sonata dependency with the modern standard
6. **PDF output** — OS handles printing
7. **.ngl file format** — read support mandatory (N103/N105/N106), write support desired

## What to KEEP (translate faithfully)

- Score data model: the object hierarchy (HEADER, SYNC/NOTE, MEASURE, STAFF, etc.)
- Engraving/layout engine: beam angles, spacing, note placement, slur curves, etc.
- The ANOTE struct and all its L/G/P domain annotations (see NObjTypes.h)
- MIDI export
- The palette-based editing paradigm

## What to CUT (delete aggressively)

- All Carbon/QuickDraw UI code
- Mac Sound Manager / OMS / FreeMIDI playback
- QuickTime integration  
- Platform-specific printing
- Don's experimental features (anything in MiscellaneousCode/ or marked experimental)
- Resource fork dependencies (.rsrc files)
- Xcode project files (NightingalePPC.xcodeproj, NightingaleIntel.xcodeproj)

## What to ADD

- MusicXML import/export
- SMuFL font support
- Retina/HiDPI display
- Cross-platform (macOS, Linux, Windows via Flutter)

## Key Data Types (from Precomps/)

The core coordinate system uses DDIST (1/16 point resolution):
```
DDIST    = short    (range ±2048 points, resolution 1/16 point)
STDIST   = short    (range ±4096 staffLines, resolution 1/8 staffLine)
SHORTQD  = SignedByte (range ±32 staffLines, resolution 1/4 staffLine)
STD_LINEHT = 8     (STDIST units per staff interline space)
```

Object types use OBJECTHEADER (linked list with position, selection, visibility) and
SUBOBJHEADER (staff number, subtype, selection). The ANOTE struct is the most complex,
with ~30 fields spanning Logical, Graphical, and Performance domains.

## File Format Versions

- N103: Legacy (files from ~2002). Smaller DOCUMENTHDR.
- N105: Current (Nightingale 5.6). Larger header with expanded font table.
- N106: Proposed future format (see NgaleFileFormatStatus.txt and NTypesNEW.h)

Previous reverse engineering produced a working NGL→MusicXML converter in Python
(see conversation history). The `4B 40` note record marker pattern and 30-byte records
were identified. String pool uses `02 <length> <string bytes>` format.

## N105 Struct Alignment (mac68k pragma)

**CRITICAL**: The N105 header (`NObjTypesN105.h` line 6) uses `#pragma options align=mac68k`.
Under mac68k alignment, **every struct is padded to a 2-byte boundary**, including single-byte
structs like `KSITEM_5` (which becomes 2 bytes on disk despite having only 1 byte of data).
This means manual byte-offset calculations from the C struct definitions will be WRONG unless
you account for this padding.

### KSITEM_5 is 2 bytes, not 1

```c
typedef struct { char letcode:7; Boolean sharp:1; } KSITEM_5;
// sizeof(KSITEM_5) = 2 (1 byte data + 1 byte mac68k padding)
```

Therefore `WHOLE_KSINFO_5` = 7 x 2 (KSItem array) + 1 (nKSItems) = **15 bytes**.
(The NBasicTypesN105.h comment at line 37 confirms: "the macro takes 15 bytes".)

### ASTAFF_5 on-disk layout (50 bytes)

```
Offset  Size  Field
------  ----  -----------------
0       2     next (LINK)
2       1     staffn
3       1     selected:1+visible:1+fillerStf:6
4       2     staffTop (DDIST)
6       2     staffLeft (DDIST)
8       2     staffRight (DDIST)
10      2     staffHeight (DDIST)
12      1     staffLines
13      1     [PADDING — align fontSize]
14      2     fontSize (short)
16      2     flagLeading (DDIST)
18      2     minStemFree (DDIST)
20      2     ledgerWidth (DDIST)
22      2     noteHeadWidth (DDIST)
24      2     fracBeamWidth (DDIST)
26      2     spaceBelow (DDIST)
28      1     clefType
29      1     dynamicType
30      14    KSItem[0..6] (7 x 2 bytes: data byte + mac68k pad byte)
44      1     nKSItems
45      1     timeSigType
46      1     numerator
47      1     denominator
48      1     filler:3+showLedgers:1+showLines:4
49      1     [PADDING — struct aligned to 2-byte boundary]
              TOTAL: 50 bytes
```

### General rule for N105 subobject unpackers

When computing byte offsets for any N105 struct, always verify against the file's
`heap.obj_size` value. If manual offset calculation disagrees with obj_size, there is
padding. Use raw hex dumps of known-good data to locate fields empirically.

## Progress & Roadmap

See `PROGRESS.md` for phase plan, current status, and next steps.

## Task Execution Pattern

Each session should:
1. Read this CLAUDE.md
2. Check `PROGRESS.md` for current state
3. Pick the next incomplete task
4. Do the work, commit, update PROGRESS.md
5. Keep commits small and focused
6. Run `cargo fmt` and `cargo clippy` before committing

## Autonomous Work Cycle

When working on rendering/engraving tasks, operate in autonomous micro-cycles:

1. **Research** — Read the OG Nightingale C source for the relevant function
   (use subagents with haiku/sonnet for grep/read tasks to conserve credits).
2. **Port** — Faithfully translate the C logic to Rust with reference comments.
3. **Render** — Run the test that produces PDF, convert to PNG, visually inspect.
4. **Self-check** — Look for collisions, misalignments, incorrect positioning.
   If issues found, iterate (go to step 1 for the next discrete issue).
5. **Checkpoint** — After fixing a batch of discrete issues, present a visual
   review to the user before committing. Ask for human feedback on bigger-picture
   direction/decisions, not micro-level code choices.

**Minimize prompts, maximize self-feedback.** The user prefers fewer back-and-forth
exchanges with more work done per cycle.

## Visual Review & Bitmap Regression

### Golden bitmaps

Every NGL and Notelist fixture has a golden bitmap in `tests/golden_bitmaps/`:
- NGL: `{fixture_name}_page1.png`
- Notelist: `nl_{fixture_name}_page1.png`

Bitmap regression tests run as part of `cargo test`. On mismatch the test panics
with the pixel diff percentage and a path to the diff image.

### Reviewing visual changes

```sh
# Quick visual diff of all golden bitmaps vs git HEAD:
./scripts/visual-review.sh

# Or run the Rust test directly:
cargo test --test golden_diff -- --nocapture

# Diff images go to test-output/golden-diff/
#   {name}_old.png   — committed version
#   {name}_new.png   — current version
#   {name}_diff.png  — visual diff (matching=dimmed, changed=red)
```

### Updating goldens after intentional rendering changes

```sh
# Regenerate NGL goldens:
REGENERATE_REFS=1 cargo test test_all_ngl_bitmap_regression

# Regenerate Notelist goldens:
REGENERATE_REFS=1 cargo test test_all_notelists_bitmap_regression

# Then review the diffs before committing:
./scripts/visual-review.sh
```

### Shared test utilities

`tests/common/mod.rs` provides `pdf_to_png()` and `compare_images_and_diff()`
used by ngl_all.rs, notelist_all.rs, and golden_diff.rs. Do not duplicate these.

## Porting Drawing Code

OG Nightingale has two separate rendering pipelines:
- **Screen**: QuickDraw-based (DrawHighLevel.cp, DrawObject.cp, DrawNRGR.cp)
- **Print/PostScript**: PS_Stdio.cp primitives (moveto, lineto, curveto, stroke, fill)

Our Rust port targets the **PostScript/print pipeline** via the MusicRenderer trait,
which maps to PDF output. When porting drawing functions, prefer the PS_Stdio.cp
code path over QuickDraw code paths. Keep the two pipelines conceptually separate
during porting; consolidation can happen later once porting is more complete.

Each discrete drawing function should be ported as its own unit with tests:
- Port one function at a time (e.g., AccXOffset, DrawLedgerLines, CalcYStem)
- Include OG source file + line number in comments
- Test with real Notelist data rendered to PDF
- Create `#[ignore]` tests for known-punted items so they show up in test output

## SMuFL Metadata Integration (Line Width Scaling)

The rendering pipeline uses **SMuFL (Standard Music Font Layout) metadata** from the Bravura font
to dynamically compute line widths based on staff height, replacing hardcoded thickness values.

### Motivation

The OG Nightingale hardcodes line thicknesses as percentages of line space (`8%`, `13%`, etc.),
which only work for one specific staff size. SMuFL metadata provides relative thickness values
(in staff spaces) that scale correctly across different staff sizes and fonts.

### Implementation

**File**: `src/smufl_metadata.rs` (240 lines)

Data structures (serde deserialization):
```rust
#[derive(Deserialize)]
pub struct SmuflMetadata {
    pub font_name: String,
    pub font_version: f32,
    pub engraving_defaults: EngravingDefaults,
    // ... glyph data (for future use)
}

pub struct EngravingDefaults {
    pub staff_line_thickness: f32,      // 0.13 staff spaces (Bravura)
    pub leger_line_thickness: f32,      // 0.16 staff spaces
    pub stem_thickness: f32,            // 0.12 staff spaces
    pub thin_barline_thickness: f32,    // 0.16 staff spaces
    // ... other thickness values
}
```

**Key Methods**:
- `SmuflMetadata::load(path)` — Load JSON from file, deserialize via serde_json
- `compute_line_widths_pt(staff_height_pt)` — Convert staff-space values → points

**Unit Conversion** (critical for correctness):
```
staff_space = staff_height_pt / 4.0              // 4 spaces per 5-line staff
line_width_pt = thickness_spaces * staff_space
```

Example: For a 24pt staff:
- staff_space = 24 / 4 = 6pt
- stem_width = 0.12 * 6 = 0.72pt

### Integration Point

**File**: `src/draw/draw_high_level.rs:42-62` (render_score function)

When rendering begins, load metadata and compute line widths dynamically:
```rust
let (staff_lw, ledger_lw, stem_lw, bar_lw) =
    if let Ok(metadata) = SmuflMetadata::load("assets/fonts/bravura_metadata.json") {
        metadata.compute_line_widths_pt(lnspace * 4.0)  // lnspace is already in points
    } else {
        // Fallback to OG defaults (percentages)
        (0.08 * lnspace, 0.13 * lnspace, 0.08 * lnspace, 0.10 * lnspace)
    };

renderer.set_widths(staff_lw, ledger_lw, stem_lw, bar_lw);
```

Graceful degradation: If Bravura metadata file is missing or corrupted, the fallback to OG
percentage-based defaults ensures rendering still works.

### Why This Matters

1. **Correct scaling** — Staff sizes now render with proportionally correct line widths
2. **Font independence** — Future font support (e.g., SMuFL fonts other than Bravura) can
   provide their own metadata without code changes
3. **Standards compliance** — Uses official W3C SMuFL specification
4. **Future extensibility** — Metadata also contains glyph bounding boxes, advance widths,
   and anchors (currently unused but available for precise glyph positioning)

### Bravura Metadata Source

- **File**: `assets/fonts/bravura_metadata.json` (716KB, v1.392)
- **Source**: https://github.com/steinbergmedia/bravura (Steinberg official)
- **Spec**: https://w3c.github.io/smufl/latest/specification/font-specific-metadata.html

## OG Nightingale Source Location

The OG C source is a **separate checkout of https://github.com/AMNS/Nightingale**,
branch `develop`. It is NOT part of this repo — it lives alongside it as a sibling
directory named `OGNGale_source/`.

### Bootstrapping (session start)

At the beginning of each session, locate the OG source before reading any OG files:

```sh
# Typical layout — repo and OG source share a parent directory:
#   <parent>/
#     nightingale-modernize/   ← this repo (pwd)
#     OGNGale_source/          ← OG C source
ls ../OGNGale_source/src/CFilesBoth/    # confirms location
```

If the `OGNGale_source/` sibling is not found, use the bootstrap script:

```sh
scripts/bootstrap-og-source.sh            # clone (branch: develop)
scripts/bootstrap-og-source.sh --update   # pull latest if already cloned
```

### Key source directories (relative to OG repo root)

```
src/CFilesBoth/    — core drawing/layout/engraving (primary porting reference)
src/CFilesEditor/  — editor operations
src/Utilities/     — utility functions
src/Precomps/      — header files / type definitions
```

Use Read/Grep tools with the discovered path. Line endings are Unix LF —
no preprocessing needed. Always verify the path resolves before reading files.

## Test Data Strategy

Primary test data sources (in priority order):
1. **Notelist files** (`tests/notelist_examples/`) — simplest, most portable
2. **NGL fixture files** (`tests/fixtures/`) — real Nightingale documents
3. **VexFlow examples** — consider translating simple ones to Notelist format
4. **MusicXML** — future import path; see icebox code and seiso.com converters
   (nl2xml: https://www.seiso.com/nl2xml/, xml2nl: https://www.seiso.com/xml2nl/)

## Subagent Usage

Use the right model for the job:
- **haiku**: File search, grep, simple code reads, quick lookups
- **sonnet**: Code analysis, moderate complexity research
- **opus**: Complex porting decisions, architecture, nuanced code translation

Launch multiple subagents in parallel when tasks are independent.

**IMPORTANT:** Always instruct subagents to use only the Grep and Read tools for
searching/reading files — never Bash with grep, tr, cat, awk, etc. Bash commands
that aren't in the auto-approved list will produce blocking permission prompts that
require manual user approval, which defeats the purpose of autonomous subagents.

**NO PYTHON:** Do not use Python scripts for data analysis, hex dumps, or computation.
Use Rust tests and approved Bash commands (cargo, xxd, etc.) instead. If a new Bash
command is needed, ask the user to add it to the allowed list rather than using an
unapproved command that will block on permissions.

## Cost Efficiency Plan

### Biggest cost drivers (observed)

1. **Opus for mechanical work** — Regression test updates (INSTA_UPDATE, REGENERATE_REFS,
   hash copy-paste) don't need Opus. Use the cheapest path that works.
2. **Context bloat from stale background shells** — Each tool call's response includes
   reminder text for every open background shell. With 20+ stale shells from prior sessions,
   every interaction burns context on irrelevant reminders.
3. **Polling loops while waiting for builds** — Repeatedly checking `BashOutput` while
   `cargo test` runs (~50s) generates multiple Opus round-trips that do nothing useful.
4. **Reading large files in full** — Reading entire files (e.g., ngl_all.rs at 700+ lines)
   when only a specific section is needed wastes context window.
5. **Session continuations** — Context summaries from prior sessions are large. Keep
   session scope focused to avoid needing many continuations.

### Rules to follow

1. **Batch rendering changes** — When a change affects multiple regression test layers,
   chain the updates into a single `&&`-connected Bash command:
   ```sh
   INSTA_UPDATE=always cargo test --test ngl_all test_all_ngl_regression_snapshots && \
   REGENERATE_REFS=1 cargo test --test ngl_all test_all_ngl_command_stream_hashes && \
   REGENERATE_REFS=1 cargo test --features visual-regression --test ngl_all test_all_ngl_bitmap_regression
   ```
2. **Run builds in background, don't poll** — Use `run_in_background: true` for long
   builds/tests. Do other work while waiting. Check output once, not in a loop.
3. **Use line ranges when reading** — Always use `offset`/`limit` when reading files
   where you know the relevant section. Don't read 700 lines to find a 5-line hash.
4. **Delegate research to subagents** — OG source exploration, grep-heavy research,
   and file discovery should use Haiku/Sonnet subagents, not Opus.
5. **Batch related file changes** — When updating hash values in ngl_all.rs, read
   the test output to get ALL changed hashes at once, then make all edits in one pass.
   Don't update one hash, re-run, discover another changed, repeat.
6. **Pre-commit hook runs full test suite** — Don't run `cargo test --all` manually
   before committing. The pre-commit hook does it. Run only targeted tests during
   development (e.g., `cargo test --test ngl_all test_all_ngl_command_stream_hashes`).
7. **Minimize BashOutput checks** — When a background command is expected to take 30+
   seconds, don't check it every 5 seconds. Wait, then check once.

### Model selection guide

| Task | Model | Examples |
|------|-------|---------|
| File search / grep | Haiku subagent | Find OG C function, locate test fixture |
| Read & summarize OG source | Sonnet subagent | Understand DrawRPTEND logic |
| Simple code edits | Haiku (if available) or direct | Hash updates, import additions |
| Port C→Rust algorithm | Opus (main) | Beam slope, spacing, slur curves |
| Architecture decisions | Opus (main) | Module organization, API design |
| Debug failing tests | Sonnet subagent for research, Opus for fix | |

## Module Map (mirrors OG C source files)

Shared algorithm modules (used by both NGL and Notelist pipelines):
```
src/
├── pitch_utils.rs     <- PitchUtils.cp  (pitch→staff position)
├── utility.rs         <- Utility.cp     (calc_ystem, nflags, std2d, head_width)
├── music_font.rs      <- MusicFont.cp   (font metrics)
├── objects.rs         <- Objects.cp     (stem direction, chord processing, key sig)
├── beam.rs            <- Beam.cp        (beam slope computation)
├── space_time.rs      <- SpaceTime.cp   (duration-proportional spacing)
```

Drawing modules (split from score_renderer.rs):
```
src/draw/
├── draw_high_level.rs <- DrawHighLevel.cp  (render_score main loop)
├── draw_object.rs     <- DrawObject.cp     (staff, measure, connect, clef, keysig, timesig, ties)
├── draw_nrgr.rs       <- DrawNRGR.cp       (sync/notes/rests, ledger lines)
├── draw_utils.rs      <- DrawUtils.cp      (glyph mapping, KS Y offsets)
├── draw_beam.rs       <- DrawBeam.cp       (beam sets)
├── draw_tuplet.rs     <- Tuplet.cp         (tuplet brackets/numbers)
├── helpers.rs         (d2r_sum, count_staves, TieEndpoint)
└── score_renderer.rs  (backward-compat re-export shim)
```

## Conventions

- Rust code: `snake_case`, modules mirror the original C file organization
- Tests: every ported function gets at least one test with data from a real .ngl file
- Comments: when porting, include a reference to the original C file and line number
- Commits:
  - Keep commits small and focused on one change
  - **DO NOT** make commits with only progress/documentation updates (ROADMAP.md, SESSION_SUMMARY.md, etc.)
  - Roll documentation updates into substantive commits (e.g., commit code + test + doc updates together)
  - Use descriptive commit messages that explain what and why

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AMNS)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AMNS)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
