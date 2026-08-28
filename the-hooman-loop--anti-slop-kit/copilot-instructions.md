## anti-slop-kit

> This repository is a portable, agent-agnostic kit for using AI without producing

# Agent instructions for anti-slop-kit

This repository is a portable, agent-agnostic kit for using AI without producing
generic, unverified slop. The canonical content lives in `skills/`. Everything
else is either an adapter for a specific tool or supporting docs.

## Source of truth

- `skills/*/SKILL.md` are the canonical Agent Skills. Follow the
  [Agent Skills spec](https://agentskills.io): a folder with a `SKILL.md` that
  has YAML frontmatter (`name`, `description`) and a markdown body.
- `adapters/` holds thin, tool-specific copies or pointers (Cursor rules, Claude
  install notes). When a skill changes, update the matching adapter so they do
  not drift.
- `prompts/` are tool-agnostic prompt cards and a demo script.

## Rules for editing this repo

- Keep it tool-neutral. Do not turn this into a Cursor-only product; Cursor is
  one install path among several.
- Do not copy third-party skill files into this repo. Absorb ideas and rewrite
  them in this kit's own voice, and credit the originals in the README's
  "Related skills" section.
- Practice what the kit preaches: specific over generic, grounded over
  impressive, humans own judgment. Apply `deslop-writing` to this repo's own
  prose.

## Layout

```
skills/       canonical Agent Skills (the product)
prompts/      copy-paste prompt cards + demo script (any tool)
adapters/     cursor/ rules, claude/ install notes
```

---
> Source: [the-hooman-loop/anti-slop-kit](https://github.com/the-hooman-loop/anti-slop-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
