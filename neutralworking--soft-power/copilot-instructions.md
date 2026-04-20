## soft-power

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Soft Power** is a multi-format creative project centered on a modern epic poem modelled on Dante's *Divine Comedy* and T.S. Eliot's *The Waste Land*. It follows Jolyon "Joly" Dox through Camden's music scene across three acts (Maundy Thursday, Good Friday, Holy Saturday) in ten scenes. From the poem, three adaptations have grown: original songs, a staged play (performed at The Nines, sold out, 2017), and a game concept called *Unreal Albion*.

**The poem is a loop.** The ending feeds back into the opening line. The cast fake their deaths; the narrator reconstructs the last three days.

The working text is **01 - Poem/soft_power_v2.md** (markdown, editable). The docx version (`soft_power_v2.docx`) is kept for export/print but the markdown is the source of truth. The earlier V1 PDF and "Objective-C or Relative" format are superseded.

## Repository Structure

This is an **Obsidian vault**, not a software repo. No git at root level. The `game/` subdirectory has its own git repo (Ren'Py visual novel).

### Numbered folders (canonical organization)

| Folder | Contents |
|--------|----------|
| `01 - Poem/` | Source poem text (Parts I & II), rewrites, research/themes, V2 docx |
| `02 - Music/` | Lyrics, music production files, Rufio Renton material |
| `03 - Play/` | Theatre scripts, production docs, rehearsal materials |
| `04 - Game/` | Unreal Albion campaign planning, scene breakdowns, mechanics (D&D-style RPG) |
| `05 - Characters/` | Hero classes, main cast, supporting characters |
| `06 - World/` | Locations, references & inspirations |
| `07 - Assets/` | Images, video |

### Legacy folders (older organization, overlapping content)

`Characters/`, `Documents/`, `Inspiration/`, `Locations/`, `Production/`, `References/`, `Rewrites/`, `Songs/`, `Text/`, `Rufio Renton/`, `Unreal Albion/` — these predate the numbered structure and contain overlapping material. The numbered folders are the canonical source.

### Key standalone files

- `ANALYSIS.md` / `ANALYSIS 2.md` — detailed structural analysis of the poem, scene-by-scene breakdown, editorial recommendations (Act Two pacing issues, character integration)
- `SOFT POWER - DRAFT MANUSCRIPT V1.pdf` — definitive poem manuscript
- `SP2 1.0.pdf` — Part II manuscript

### Unreal Albion — Godot 4 Game (`04 - Game/unreal-albion/`)

A narrative RPG with musical performance combat. Vertical slice: "The Albion: Night One" — a 20-30 minute evening at a Camden music venue.
- **Engine:** Godot 4 (GDScript)
- **Core systems:** Ego/Creativity/Attention three-stat economy, JSON-driven dialogue, turn-based Stage Confrontation combat, Attention Economy (shared Room Heat pool)
- **Run via:** Open `04 - Game/unreal-albion/project.godot` in Godot 4.3+
- **Key directories:** `autoloads/` (5 singletons), `scenes/locations/` (6 scenes), `ui/` (HUD, dialogue, phone, combat UI), `data/` (dialogue JSON, enemy data, passive inserts)
- **Cast:** Ash Vale (PC), Mara Flint, Jules Rook, Viv Kane, Bootsy
- **Three endings:** Breakthrough (survived with integrity), Compromise (sold out), Fold (Ego zero — became an NPC)

### Legacy Ren'Py Game (`Soft Power/game/`)

An earlier visual novel attempt (abandoned). Empty directory with error logs. Superseded by the Godot project above.

### Unreal Albion (D&D Campaign)

A tabletop RPG adaptation with custom mechanics: **Ego** (HP), **Creativity** (Mana), **Attention** (Economy/currency). The SRD is embedded in the Brennan Mulligan skill (`.claude/skills/brennan-mulligan/SKILL.md`).

## Narrative Structure

**Act One — Maundy Thursday:** Scenes 1-4 (Sunset wake → Norah's train → Burial/Ruckenfigur debut → Poets' kitchen, WK vanishes)
**Act Two — Good Friday:** Scenes 5-7 (Canal soliloquy → Ron's office/Destiny → Norah walks to The Albion)
**Act Three — Holy Saturday:** Scenes 8-10 (Albion rap battle/Cosmo reveal → Time-split Tom/eulogy → Final gig, fire, loop)

## Key Characters

- **Joly Dox** — protagonist, producer, grieving brother
- **Cosmo Dox** — Joly's brother, faked death at Port of Spezia, revealed as Ruckenfigur
- **Norah Healy** — strongest character per analysis, kintsugi speech, reads Cosmo's eulogy
- **Ron Dox** — father/antagonist, "I feel no guilt" monologue
- **Kwame Kofi Kinsey** — Oxford dropout, poet-programmer
- **WK Blue** — American rapper, vanishes in power cut
- **Delphine $appho** — Joly's ex-wife
- **Destiny Vegas** — real name Anna Bartholomew, currently isolated from plot
- **Tom Eliot-White** — time-split scene in Act Three

## Known Editorial Issues (from ANALYSIS.md)

- **Act Two pacing:** Three character studies where the poem needs plot mechanics; recommendation is to tighten each scene to its strongest thread
- **Destiny Vegas** has no interaction with other characters — needs integration or acceptance as thematic portrait
- **Yoshikoko** appears once but a "pact" creates unfulfilled expectation
- **Narrator naming:** "NARRATOR" everywhere except one "TYRESE" in Fire Sermon — needs consistency
- **Loose verse** in Fire Sermon reads more like essay than poetry

## Working with This Project

- Markdown files use **Obsidian** wiki-link syntax (`[[links]]`, `![[embeds]]`, callout blocks `> [!info]`)
- `.rpyc` files in `game/` are compiled bytecode — never edit them
- The `assets/` folder contains photos, videos, and a Python credits script (`objectivecorrelative.py`)
- When editing poem text or rewrites, preserve the four-line stanza structure used in V1

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neutralworking) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
