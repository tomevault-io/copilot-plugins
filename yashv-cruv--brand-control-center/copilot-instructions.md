## brand-control-center

> Skill at `.claude/skills/storybook/SKILL.md`. Invoke with `/storybook` to generate stories for any component.

## storybook

Skill at `.claude/skills/storybook/SKILL.md`. Invoke with `/storybook` to generate stories for any component.

Covers: placement rules, title convention, template, path aliases, \_utils helpers, mock API patterns, dark mode, quality checklist (codex validation criteria).

## graphify

This project has a graphify knowledge graph at graphify-out/.

Rules:

- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- For cross-module "how does X relate to Y" questions, prefer `graphify query "<question>"`, `graphify path "<A>" "<B>"`, or `graphify explain "<concept>"` over grep — these traverse the graph's EXTRACTED + INFERRED edges instead of scanning files
- After modifying code files in this session, run `graphify update .` to keep the graph current (AST-only, no API cost)

---
> Source: [yashv-cruv/Brand-Control-Center](https://github.com/yashv-cruv/Brand-Control-Center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
