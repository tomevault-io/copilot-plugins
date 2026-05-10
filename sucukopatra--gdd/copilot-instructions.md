## gdd

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A Game Design Document (GDD) for an unnamed narrative puzzle game, authored in [Typst](https://typst.app/). The compiled output is `gdd.pdf`.

## Build command

```sh
typst compile gdd.typ
```

This regenerates `gdd.pdf` from `gdd.typ`. Typst must be installed (`typst` CLI).

## Document structure

The entire document lives in `gdd.typ`. It is organized into top-level sections using Typst heading syntax (`==` for h2, `===` for h3, `====` for h4):

1. **Game Overview** — genre, platform, audience
2. **Setting & Narrative** — story, characters
3. **Game Logic** — core loop, perspective switching
4. **Gameplay Systems** — Scientist Phase and Virus Cube Phase mechanics
5. **Mini-Games** — four distinct mini-games (Isometric Adventure, Text Adventure, Top-Down Racing, Terminal Login Puzzle)
6. **Level Structure** — per-corridor asset lists
7. **HUD Design** — per-phase UI specifications
8. **Audio Design** — ambient layers, interaction sounds, music system
9. **Art Direction** — visual style, CRT treatment, reference works

Images are stored in `images/` and referenced inline. `SonicDesign.pdf` is a supplemental asset, not part of the main document.

## Typst conventions used in this file

- `#terminal(body)` — custom block styled as a green-on-black terminal readout; used for HUD examples
- `#pagebreak()` — explicit page breaks between major sections
- `#figure(image(...), caption: "...", numbering: none)` — captioned images with numbering suppressed
- `#grid(columns: 2, ...)` — two-column character/asset grids
- `#stack(dir: rtl/ltr, ...)` — side-by-side image layouts

---
> Source: [sucukopatra/gdd](https://github.com/sucukopatra/gdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
