## openknowledge

> This repository has a colocated Open Knowledge wiki at `Wiki/`.

# Repository Agent Instructions

This repository has a colocated Open Knowledge wiki at `Wiki/`.

Use the repo skill at `.codex/skills/openknowledge-wiki/SKILL.md` when touching Open Knowledge CLI behavior, command flags, exporters, validation, the local viewer, setup flow, README or docs that describe CLI behavior, or release-impacting package changes.

Before making those changes, read:

* `Wiki/AGENTS.md`
* `Wiki/workflows/feature-docs.md` for command or feature documentation updates
* `Wiki/workflows/changelog-updates.md` for CLI changelog updates

After meaningful wiki edits, run:

```sh
okn validate "Wiki"
```

Do not update the wiki for unrelated formatting-only edits or changes that do not affect CLI behavior, docs, workflows, or release-facing behavior.

---
> Source: [openknowledge-sh/openknowledge](https://github.com/openknowledge-sh/openknowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
