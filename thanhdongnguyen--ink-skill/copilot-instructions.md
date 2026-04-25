## ink-skill

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A **skills package** for the [Ink](https://github.com/vadimdemedes/ink) CLI framework, distributed via `npx skills add thanhdongnguyen/ink-skill`. It contains no application code — only documentation reference files and skill manifests consumed by AI coding assistants.

## Repository Structure

```
skills/ink/
├── SKILL.md              # Main skill manifest with decision trees and product index
└── references/           # Reference docs organized by concern
    ├── core/             # 5-file pattern: REFERENCE.md, api.md, configuration.md, patterns.md, gotchas.md
    ├── components/       # REFERENCE.md + text.md, box.md, utilities.md
    ├── hooks/            # REFERENCE.md + input.md, app-lifecycle.md, stdio.md, focus.md
    ├── layout/           # REFERENCE.md + patterns.md
    ├── testing/          # REFERENCE.md
    └── accessibility/    # REFERENCE.md

command/ink.md            # /ink command entrypoint (loaded when user runs /ink)
```

## Key Conventions

**5-file pattern for core references** — each major area under `references/core/` follows: `REFERENCE.md` (overview + quick start), `api.md` (API surface), `configuration.md` (options/env vars), `patterns.md` (recipes), `gotchas.md` (pitfalls).

**SKILL.md is the entry point** — it contains decision trees that route to the correct reference files. When adding new reference content, update the decision trees and product index in `SKILL.md`.

**command/ink.md is the `/ink` command** — it tells the AI how to load the skill and which reference files to read based on the user's task. Update this when new task types are added.

**No build step** — all files are markdown. No package.json, tests, or CI pipeline.

## When Editing Reference Files

- Keep each reference file focused on its stated concern
- `REFERENCE.md` files are entry points — always read first, so keep them concise with links to detail files
- Cross-cutting concepts (layout, testing, accessibility) are single-file guides, not 5-file patterns
- Ink critical rules (all text in `<Text>`, `<Box>` is always flex, use `useApp().exit()`) live in `SKILL.md` — do not duplicate them across reference files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thanhdongnguyen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
