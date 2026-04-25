## lp-lineage-theme-capture

> This repository is a **git submodule** of the [plonetheme.lp](https://github.com/sdaitzman/lp) theme codebase. It is not intended to be worked on independently — it provides reference material and workflow documentation consumed by the parent repo.

# CLAUDE.md — lp-lineage-theme-capture submodule

This repository is a **git submodule** of the [plonetheme.lp](https://github.com/sdaitzman/lp) theme codebase. It is not intended to be worked on independently — it provides reference material and workflow documentation consumed by the parent repo.

## What's here

- **`captured-themes/`** — Per-sub-site crawled HTML, CSS, screenshots, and style guides from `landscapepartnership.org`
- **`plans/`** — Repeatable workflow documents for theme replication

## Key file: `plans/subsite-theme-replication.md`

This is the master workflow for taking a captured sub-site and replicating its visual design in `plonetheme.lp`. It covers crawling, SCSS scaffolding, structural audits, incremental style adoption, print styles, and validation.

## Working with this submodule

When making changes here (adding captures, updating plans, etc.):

1. Commit inside this submodule first
2. Then commit the updated submodule reference in the parent repo

```bash
# From within this directory
git add .
git commit -m "Description of change"

# Then from the parent repo root
cd ..
git add lp-lineage-theme-capture
git commit -m "Update theme capture submodule"
```

## Path conventions

All paths in `plans/subsite-theme-replication.md` are written **relative to the parent repo root** (e.g. `lp-lineage-theme-capture/captured-themes/SITE-styles/`), since that plan is executed from the parent repo context.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sdaitzman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
