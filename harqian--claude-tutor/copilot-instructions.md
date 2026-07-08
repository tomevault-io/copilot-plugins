## claude-tutor

> Project-specific rules for `~/code/ai_tutor`. Global rules in `~/AGENTS.md` apply.

# AGENTS.md

Project-specific rules for `~/code/ai_tutor`. Global rules in `~/AGENTS.md` apply.

This is Harrison's personal AI-tutor system. Project overview in `README.md`; read it first before changing structure.

## Core conventions

- **Goal function.** The tutor's job is to leave the learner more capable, not to resolve questions. When in doubt, withhold; ask before telling.
- **21 principles.** All session behavior derives from `system/prompts/core.md`. Every mode skill loads it.
- **Modes are Claude skills, packaged as the `ai_tutor` plugin.** Skills live at `skills/<mode>/SKILL.md`. `.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json` make this a local Claude Code plugin; `.claude/settings.json` enables it whenever cwd is in this repo. Skills load with the `ai_tutor:` namespace (e.g. `/ai_tutor:socratic`, `/ai_tutor:curriculum-research`) so all available modes are visible at a glance in the slash-command list. When adding a new mode: write `skills/<name>/SKILL.md`, then `/reload-plugins`. No symlinks, no manual install. Do not invent a custom mode loader.
- **Curriculum-research before tutoring.** New domain? Run the `curriculum-research` skill first to produce `target.yaml` + `curriculum.yaml`. It declares its confidence bucket (canonical / fuzzy / niche) up front and routes source-curation to Harrison's taste rather than guessing.

## File and data conventions

- **Provenance.** Every LLM-authored file (yaml, md) gets frontmatter:
  ```yaml
  generated_by: claude-opus-4-7
  generated_at: 2026-05-03
  ```
  No human-review flag. Harrison reads what he reads; tracking it is overhead with no consumer.
- **Mastery is authored.** `domains/<domain>/mastery.yaml` is a living document: free-text summary, per-concept state (unseen/shaky/solid), plus a selective log of sessions worth remembering. Updated each session, not appended. Routine drills update state and disappear; only insight moments / decisions / mode shifts get a log entry.
- **Cards live in Mochi, but the inventory lives in `mastery.yaml`.** Push via the `mochi` MCP. Never create a separate `cards.yaml`. Whenever a session creates cards, the session's `log` entry **must** include a `mochi_cards` block with id, deck, front, back for each card. This is the canonical card inventory — without it we lose track of what we've made. Tag cards in Mochi by domain and concept slug for cross-reference.
- **Always load the `mochi` skill before drafting or pushing cards.** It is the design canon (six commitments, T1–T10 templates, anti-patterns, self-check). Pattern-matching off existing cards is not a substitute. The skill's auto-fire on trigger phrases ("make a mochi card", "push to mochi") will miss when card creation is a follow-up step in a larger task, so load it explicitly via the `Skill` tool whenever cards are in scope.
- **Notes are flat per-domain.** `domains/<domain>/notes/<concept-slug>.md`. No per-concept folders. These are LLM-authored study notes (carry frontmatter).
- **Harrison's own per-domain writing lives at `domains/<domain>/scratch/`.** Free-form spitball space alongside (not inside) the LLM-authored `notes/`. Not LLM-authored, no frontmatter required, any filenames Harrison wants. The tutor reads these for context but does not write to them unless asked. Use this for the kind of running-doc / roadmap / reflection writing Harrison does for himself while learning a domain.
- **Prereqs are soft.** Edges in `curriculum.yaml` are advisory. No blocking-graph traversal.

## Visualizations

- **Never overwrite.** Each viz gets `out/artifacts/YYYY-MM-DD_HHMMSS_<topic-slug>.html`. Full history preserved.
- **Self-contained HTML.** CDN imports, inline JS, no external assets.
- **`live-server` runs in `out/`.** Tell user `localhost:5500/artifacts/<filename>` after writing.
- If `live-server` is not installed as a direct binary, run `npx -y live-server --port=5500 --host=127.0.0.1 --no-browser` from `out/` in tmux.
- **Library defaults and craft principles** in `system/visualization.md`. Read it before generating viz.

## Session shape

Default flow when Harrison opens a session:

1. Load `student/profile.yaml` for calibration.
2. Identify domain (ask if unclear). Load `domains/<domain>/target.yaml`, `curriculum.yaml`, `mastery.yaml`.
3. Apply principle 21: elicit target structure if not in `target.yaml`.
4. Run the session in the appropriate mode (default: socratic).
5. End with: propose `mastery.yaml` update (revised summary, concept state changes, optional log entry if session is worth remembering) → Harrison approves → write. Propose Mochi cards for any insight moments → Harrison approves → push to Mochi → record id/deck/front/back in the session's `log[].mochi_cards` block. Cards without a log entry are invisible to future sessions.

## Bias toward simplicity

YAGNI applies hard here. The whole system is deliberately small: flat notes, soft prereqs, no per-concept folders, no card store. Resist adding structure until a real session surfaces a real need.

## When extending

Before adding a new file format, schema, or directory, ask: is this earning its way in? The simplifications in v0 (no per-concept files, no cards.yaml, soft prereqs) were chosen to minimize maintenance burden. Each addition has a tax.

When you do add: update `README.md` if the layout or skill list changes, update this file if the convention changes, and add a real example so the next agent has something to pattern-match against.

---
> Source: [harqian/claude-tutor](https://github.com/harqian/claude-tutor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
