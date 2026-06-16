## skidl-skills

> AI-assisted KiCad PCB design using the SKiDL Python library.

# SKiDL PCB Design Plugin

AI-assisted KiCad PCB design using the SKiDL Python library.

## Quick Start

- **New design:** `/new-circuit`
- **Design review:** `/design-review`
- **Find a part:** `/find-part`
- **Run a circuit:** `.venv/bin/python circuits/<name>.py` (set `KICAD_SYMBOL_DIR` first — see `rules/environment.md`)

## Rules

Detailed conventions and workflow documentation live in `rules/`:

| Rule | Scope | Content |
|---|---|---|
| `environment.md` | always | Python, KiCad paths (cross-platform), how to run circuits |
| `project-layout.md` | always | Directory structure |
| `design-workflow.md` | always | Pipeline stages, agents, skills, MCP tools, hooks |
| `skidl-syntax.md` | `**/*.py` | Coding conventions, syntax reference, ERC rules, footprint validation |
| `kicad-footprints.md` | `**/*.kicad_mod` | Footprint format, custom libraries |

---
> Source: [nickkraakman/skidl-skills](https://github.com/nickkraakman/skidl-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
