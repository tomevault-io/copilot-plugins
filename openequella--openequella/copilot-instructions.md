## openequella

> The shared instructions above apply to every agent. This section covers Claude Code specifics.

@AGENTS.md

## Claude Code

The shared instructions above apply to every agent. This section covers Claude Code specifics.

- **Skills** live in `.agents/skills/`, the cross-agent convention, and are exposed to Claude Code
  through the committed `.claude/skills` symlink. Add new skills under `.agents/skills/`, never under
  `.claude/skills/`.
- **Personal settings** belong in `.claude/settings.local.json`, which is gitignored. Only put
  settings the whole team should share in a committed `.claude/settings.json`.

---
> Source: [openequella/openEQUELLA](https://github.com/openequella/openEQUELLA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
