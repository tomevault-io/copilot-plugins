## vc-update-skill

> This repo ships one Agent Skill: `skills/vc-update/`. It is also discoverable at

# vc-update-skill

This repo ships one Agent Skill: `skills/vc-update/`. It is also discoverable at
`.agents/skills/` (a symlink) so Codex finds it when working inside this repo.

To use the skill here: `/vc-update` in Claude Code or Cowork, `$vc-update` in
Codex.

To work on the skill:

- The entry point is `skills/vc-update/SKILL.md`. Keep it lean; step-by-step
  detail belongs in `references/` (progressive disclosure), output templates in
  `assets/`.
- SKILL.md frontmatter must stay within the Agent Skills spec fields (`name`,
  `description`, `license`, `compatibility`, `metadata`, `allowed-tools`) so the
  skill stays portable across Claude Code, Cowork, Codex, and claude.ai uploads.
  `metadata` values must be strings.
- Do not hard-depend on any harness-specific tool. The interactive question
  tool, the Gmail connector, and the scheduler are all optional, with plain-text
  / `mailto:` / calendar fallbacks spelled out in the references.
- Two runtime invariants the references enforce: (1) never send an email without
  showing the draft and getting a yes; (2) never propose the reminder again once
  `state.json` records `reminder.status: "declined"`.
- The fund recipient lives in `SKILL.md` frontmatter (`metadata.recipient`) and
  `assets/state.example.json` — change both together.
- Bump `version` in `.claude-plugin/plugin.json`,
  `.claude-plugin/marketplace.json`, and the SKILL.md `metadata` together.

---
> Source: [cyber-fund/vc-update-skill](https://github.com/cyber-fund/vc-update-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
