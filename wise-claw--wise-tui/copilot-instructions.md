## wise-tui

> <!-- TRELLIS:START -->

<!-- TRELLIS:START -->
# Trellis Instructions

These instructions are for AI assistants working in this project.

This project is managed by Trellis. The working knowledge you need lives under `.trellis/`:

- `.trellis/workflow.md` — development phases, when to create tasks, skill routing
- `.trellis/spec/` — package- and layer-scoped coding guidelines (read before writing code in a given layer)
- `.trellis/workspace/` — per-developer journals and session traces
- `.trellis/tasks/` — active and archived tasks (PRDs, research, jsonl context)

If a Trellis command is available on your platform (e.g. `/trellis:finish-work`, `/trellis:continue`), prefer it over manual steps. Not every platform exposes every command.

If you're using Codex or another agent-capable tool, additional project-scoped helpers may live in:
- `.agents/skills/` — reusable Trellis skills
- `.codex/agents/` — optional custom subagents

Managed by Trellis. Edits outside this block are preserved; edits inside may be overwritten by a future `trellis update`.

<!-- TRELLIS:END -->

## Wise Product Evolution Rule

Wise is being productized toward an AionUi-style AI workbench. Backend changes are allowed when they support that direction, but existing backend capabilities must not be deleted. Prefer migrating, merging, or wrapping existing functionality into clearer Hub / Channel / Automation / Artifact surfaces instead of removing commands, data, or integration paths.

Every configuration-center menu must be reviewed as a product surface, not preserved as a legacy interaction. For each menu, decide how it should support the AI workbench loop, local/remote Agent supply, scheduled automation, artifact review, channel access, or runtime control. Chinese labels are preferred for built-in UI. Single-platform controls must be folded into neutral Channel or Hub surfaces instead of becoming permanent top-level entries.

---
> Source: [wise-claw/wise-tui](https://github.com/wise-claw/wise-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
