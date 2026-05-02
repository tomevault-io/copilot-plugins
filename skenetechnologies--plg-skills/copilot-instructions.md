## plg-skills

> Guidance for working in this repo (PLG Skills) with Cursor. Keeps edits consistent with the existing Claude Code skill format.


# PLG Skills (repo rules)

This repository is a **library of AI “skills” for Product-Led Growth (PLG)**. Each skill lives in `skills/<skill-name>/SKILL.md` and is designed to be used with AI assistants (see `README.md` for one install path).

When working in this repo, optimize for **clear, reusable instruction files** (not application code).

## What to preserve

- **YAML front matter**: Every `skills/*/SKILL.md` starts with front matter like:
  - `name`: must match the directory name (e.g. `skills/plg-strategy/SKILL.md` has `name: plg-strategy`)
  - `description`: one-paragraph router describing when the skill should be used; include a few example user phrases and cross-references
- **Markdown structure**: Keep headings, tables, checklists, and templates consistent and scannable.
- **Cross-references**: Prefer referencing other skills by backticks (e.g. `growth-loops`) when a topic has a dedicated skill.

## How to edit a skill

- **Maintain “role + mission” clarity** near the top (e.g. “You are a PLG strategist…”).
- **Prefer actionable artifacts**:
  - diagnostic questions
  - step-by-step frameworks
  - checklists and templates the user can copy/paste
  - metrics definitions and example calculations where relevant
- **Ground recommendations in context**: product type, buyer/user persona split, ACV, market dynamics, motion (PLG/marketing-led/sales-led), maturity stage.
- **Avoid fluff**: keep content dense, specific, and opinionated (with caveats).

## Adding a new skill

- **Create a new directory**: `skills/<new-skill>/SKILL.md`
- **Front matter requirements**:
  - `name: <new-skill>` (must match directory)
  - `description:` include routing keywords and cross-references to existing skills
- **Update `README.md`**: add the new skill to the appropriate table/category and keep the total count accurate.

## Repo-specific conventions

- This repo is **Markdown-only**; avoid introducing code unless it’s inside fenced blocks as part of a template or example.
- Prefer **consistent terminology** across skills: activation/aha moment, loops, Motions x Levers, Four Fits, S-curves, etc.
- If you change a concept that’s referenced elsewhere, consider whether related skills should be updated for consistency.

---
> Source: [SkeneTechnologies/plg-skills](https://github.com/SkeneTechnologies/plg-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
