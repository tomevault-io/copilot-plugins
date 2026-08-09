## genieparser

> These are repo-local standing instructions for genieparser work. When asked to

# Genieparser AI Agent Instructions

These are repo-local standing instructions for genieparser work. When asked to
review a genieparser PR, use the repo-local `genieparser-code-review` skill in
`.agents/skills/genieparser-code-review/`; it contains the parser expectations,
full parser-review rubric, coverage rules, regex style checks, and required
review output format.

## Agent Guidance Layout

- Keep reusable workflow playbooks in `.agents/skills/` so Codex, Copilot, and
  other compatible agents can discover the same skill shape.
- Keep `.codex/` for Codex-specific runtime configuration such as project
  config, custom agents, hooks, and command rules. Do not put shared skills
  there unless a Codex-only wrapper is needed.

---
> Source: [CiscoTestAutomation/genieparser](https://github.com/CiscoTestAutomation/genieparser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
