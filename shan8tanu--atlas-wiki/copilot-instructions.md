## atlas-wiki

> Before doing anything, read `.ai-state/STATE.md`. It contains the current folder structure, feature registry, and a log of what every previous session changed. This saves you from re-exploring the codebase.

# Atlas — Claude Code Instructions

## First: Read the State File

Before doing anything, read `.ai-state/STATE.md`. It contains the current folder structure, feature registry, and a log of what every previous session changed. This saves you from re-exploring the codebase.

## After Every Session: Update the State File

At the end of your session, append a new entry to the **Session Log** section in `.ai-state/STATE.md`. Follow the format documented in that file. Never rewrite or delete existing entries.

## Core Rules

1. **Never edit generated Markdown.** Country pages like `docs/japan.md` don't exist on disk — they're generated in-memory by `gen_pages.py`. All factual changes go in `data/visas/*.yaml`. All layout changes go in `templates/country.md.jinja`.

2. **YAML is the source of truth.** The `data/visas/` folder is the database. Every country's visa data lives there.

3. **Run `python validate.py` after editing YAML.** This catches structural errors before they break the build.

4. **Use `mkdocs serve` for local preview.** The dev server config is in `.claude/launch.json` (port 8000).

5. **Don't commit `site/` or `docs-internal/`.** Both are gitignored. `site/` is auto-generated; `docs-internal/` is local documentation.

---
> Source: [shan8tanu/atlas-wiki](https://github.com/shan8tanu/atlas-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
