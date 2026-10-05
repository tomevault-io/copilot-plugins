## skills

> This repo is a **plugin marketplace** distributing an engineering method as ten skills for

# Repository guide (Codex)

This repo is a **plugin marketplace** distributing an engineering method as ten skills for
Codex and Claude Code. It is not an app — there is no build, no dependencies, and no tests.

- The skills live in `skills/<name>/SKILL.md` — one file per skill, the single source of
  truth. Both harnesses read the same directory; there are no per-platform copies.
- Every skill holds the same three sections: `## Rules`, `## Method`, `## Output`. Rules
  run 6–9 bullets. Frontmatter is `name` + `description` only, so it works in both agents.
- `description` carries the trigger phrases *and* the routing to sibling skills — it is
  what makes a skill fire without being invoked. Edit it as carefully as the body.
- **Adding, renaming, or removing a skill touches four places**: the new `SKILL.md`, the
  routing table and disambiguation pairs in `skills/which/SKILL.md` (skill names appear
  ~27 times there), the four `description` fields across `.codex-plugin/plugin.json`,
  `.claude-plugin/plugin.json`, and `.claude-plugin/marketplace.json`, and the README's
  Reference section plus its skill count in the opening paragraph.
- The Codex manifest is `.codex-plugin/plugin.json` (`skills: "./skills/"`). Codex resolves
  the marketplace from `.claude-plugin/marketplace.json` — verified via
  `codex plugin marketplace add formesean/skills` — so there is no `.agents/` marketplace
  file to maintain.
- Bump `version` in both plugin manifests when skills change. Keep them in step.
- `CLAUDE.md` is the Claude-facing copy of this file — update both together.

Skills are standing instructions, not narration: cut prose that defends a rule rather than
stating it. The quality gate here is reading the diff — a skill is prose, so the check is
whether a model could follow it without the conversation that produced it.

---
> Source: [formesean/skills](https://github.com/formesean/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
