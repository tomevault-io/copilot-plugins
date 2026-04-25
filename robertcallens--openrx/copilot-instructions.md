## openrx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenRX is an open-source 2.4GHz ExpressLRS (ELRS) receiver hardware project. This is a **KiCad 9.0 PCB design repository** — there is no software build system. All work is done in KiCad.

## Schematic Hierarchy

- `Hardware/OpenRX.kicad_sch` — Top-level schematic (sheet container only)
- `Hardware/OpenRx24Mini.kicad_sch` — The full 2.4GHz Mini receiver design (all components live here)

## Custom Libraries

All libraries use `${KIPRJMOD}` relative paths — they resolve automatically when opening the project from `Hardware/OpenRX.kicad_pro`. No manual library setup needed.

| Library | Path | Contents |
|---|---|---|
| `SX1280` | `Symbols/SX1280.kicad_sym` | Semtech SX1280 2.4GHz transceiver |
| `OpenRX_Symbols` | `Symbols/OpenRX_Symbols.kicad_sym` | Project-specific symbols |
| `OW7EL89CENUNFAYLC-52M` | `Symbols/OW7EL89CENUNFAYLC-52M.kicad_sym` | RF module |
| `OpenRX_Footprints` | `Footprints/` | QFN24 variants, oscillator SMD package |

## Key Design Files

- `Hardware/OpenRX.kicad_pcb` — PCB layout
- `Hardware/OpenRX.kicad_pro` — Project file (open this in KiCad)
- `Hardware/3D/` — STEP and WRL models for 3D viewer

## .gitignore Notes

The following are intentionally excluded from git:
- `*.kicad_prl` — per-user window/layer state
- `*-backups/` — KiCad auto-backup folder (`OpenRX-backups/`)
- `fp-info-cache` — auto-generated footprint cache

## License

CERN-OHL-S-2.0 (Strongly Reciprocal). Any derivative hardware must also be released under CERN-OHL-S-2.0.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RobertCallens) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
