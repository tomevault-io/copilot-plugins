## icm-template

> <!-- Auto-generated from IDENTITY.md — edit IDENTITY.md to update -->

<!-- Auto-generated from IDENTITY.md — edit IDENTITY.md to update -->

# ICM — Identity

## Purpose

A skill-based distribution of the Interpretable Context Methodology for organizing AI workflows using folder structure.

---

## Folder Map

```
icm-template/
├── SKILLS
│   ├── icm-scaffold
│   │   └── SKILL.md         # The skill — generates ICM layers on any project
│   ├── icm-sync
│   │   └── SKILL.md         # Maintenance — syncs IDENTITY.md and CONTEXT.md with disk
│   └── icm-context-scaffold
│       └── SKILL.md         # Maintenance — generates missing CONTEXT.md files
├── docs
│   ├── methodology.md       # Full ICM explanation (5 layers, 5 principles)
│   ├── best-practices.md    # Writing effective stage contracts and _config/
│   ├── layer-reference.md   # Quick reference for all 5 layers
│   ├── model-adapters.md    # Per-tool setup (Claude, Cursor, Copilot, Windsurf)
│   └── Interpretable_Context_Methdology_.pdf  # Original research paper
├── IDENTITY.md              # Layer 0 — workspace map
├── CONTEXT.md               # Layer 1 — task routing
├── CLAUDE.md                # You are here
├── LICENSE                  # MIT
└── README.md                # Project overview and quick start
```

---

## Rules

- The SKILL.md files are the primary products. Everything else is supporting documentation.
- `icm-scaffold` generates ICM layers on the user's project. `icm-sync` and `icm-context-scaffold` maintain them over time.
- The skills generate files on the user's project — they do not generate files in this repo.
- Docs are educational — they explain the methodology for humans who want to understand the *why*.
- Do not add Python tools, templates, or scaffolding scripts. The skills replace all of those.

---
> Source: [ktnCodes/icm-template](https://github.com/ktnCodes/icm-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
