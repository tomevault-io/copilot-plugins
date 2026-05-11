## sim-skills

> **This file is for contributors.** If you are *using* these skills —

# CLAUDE.md — for contributors editing sim-skills

**This file is for contributors.** If you are *using* these skills —
driving a solver from your own agent or project — you do not read this
file. The Anthropic skill loader picks up `<solver>/SKILL.md` files by
their YAML frontmatter (`name` + `description`); it does not walk
surrounding `CLAUDE.md` files into an end user's session.

You only see this file if you ran Claude Code from *inside a clone of
sim-skills*. In that case you are almost certainly editing a skill,
adding a new one, or refactoring shared content. The rest of this file
is for that audience.

End-user docs live in [`README.md`](README.md) (the human-facing grid)
and in [`sim-cli/SKILL.md`](sim-cli/SKILL.md) (the agent-facing runtime
contract every driver skill depends on).

---

## What lives where

| Kind of content | Location | Audience |
|---|---|---|
| Shared runtime contract (lifecycle, commands, version probe, acceptance, escalation, input classification) | [`sim-cli/`](sim-cli/SKILL.md) + `sim-cli/reference/*.md` | End users' agents |
| Solver-specific protocol (physics, APIs, quirks per version) | The matching `sim-plugin-<name>` repo, bundled under that package's `_skills/<name>/` directory | End users' agents |
| Runtime-injected tool skills (actuation objects sim-cli puts into `sim exec` — `gui`, future `fs` / `shell` / etc.) | [`sim-cli/<tool>/SKILL.md`](sim-cli/) | End users' agents |
| Human-facing overview, skill grid, news | [`README.md`](README.md) | Humans browsing GitHub |
| Contributor guide (this file) | `CLAUDE.md` | You, right now |

**Rule of thumb while editing:** if a change belongs in more than one
driver, it belongs in `sim-cli/`. If it is solver-specific, it belongs
in the matching `sim-plugin-<name>` repo. Runtime-injected objects
(`gui`, future `fs` / `shell`, …) — anything sim-cli puts into the
`sim exec` namespace alongside `session` / `model` — belong under
`sim-cli/<tool>/`. Never duplicate across drivers.

### `sim-cli/<tool>/` — runtime-injected tool skills

Some capabilities are not a solver at all but a **runtime-injected
tool** that sim-cli exposes in the `sim exec` namespace for every
GUI-capable driver. They live under `sim-cli/` because they are
owned by sim-cli (not by any individual solver). `gui` is the first
example:

```
sim-cli/
  SKILL.md            Runtime contract (session lifecycle, commands, …)
  reference/          Shared references
  gui/                Runtime-injected actuation object
    SKILL.md          Full API — find / click / send_text / snapshot
    snippets/
      dismiss_login_dialog.py
      fill_file_save_dialog.py
      dismiss_script_error.py
```

Agents discover these via `/connect`'s `tools: [...]` + `tool_refs:
{...}` fields (e.g. `tool_refs.gui = "sim-skills/sim-cli/gui/SKILL.md"`).
Per-solver `SKILL.md` files add **short** sections pointing at
`../sim-cli/<tool>/SKILL.md` and noting solver-specific dialogs; the
API itself is not repeated per solver.

---

## When editing a skill

1. Keep the YAML frontmatter valid: `name` (letters / numbers / hyphens
   only) and `description` (starts with "Use when…", third person,
   focused on triggering conditions, NOT a workflow summary).
2. Don't move heavy reference content into `SKILL.md` — keep it in
   `reference/` and link.
3. If the change is a shared runtime concern (lifecycle, commands,
   classification, acceptance, escalation, version awareness), edit
   [`sim-cli/`](sim-cli/SKILL.md) instead of the per-driver SKILL.md.
4. Update the `## File index` section of the SKILL.md if you add or
   rename files.
5. Drift between a skill and its driver plugin is a bug. Fix one or the
   other; don't leave them disagreeing. For extracted drivers, the
   plugin repo is the source of truth for driver-specific skills.

## When adding a new solver skill

1. Create or update the matching `sim-plugin-<name>` repository.
2. Add the skill under the plugin package, normally
   `src/sim_plugin_<name>/_skills/<name>/SKILL.md`.
3. Start by pointing to this repo's [`sim-cli/SKILL.md`](sim-cli/SKILL.md)
   — the shared contract is not repeated. Keep the plugin SKILL.md
   focused on the solver overlay.
4. Register the plugin through
   [`sim-plugin-index`](https://github.com/svd-ai-lab/sim-plugin-index)
   when it is public. Commercial plugin availability depends on
   third-party license conditions; contact <contact@svd-ai-lab.com> to
   discuss commercial plugin access.
5. Add or keep a wheel-content test in the plugin repo that proves
   `_skills/<name>/SKILL.md` ships in the built wheel.
6. Do not add new solver folders to this repo. This repo is for shared
   runtime protocol and runtime-injected tool skills.

## Runtime dependency

These skills control the [`sim`](../sim-cli/) runtime. If you are
developing locally, the runtime repo sits at `../sim-cli/`. See
`../sim-cli/CLAUDE.md` for its internals (runtime protocol, plugin
loading, HTTP endpoints). Solver drivers now live in individual
`sim-plugin-<name>` repositories and are discoverable through the
plugin index.

---
> Source: [svd-ai-lab/sim-skills](https://github.com/svd-ai-lab/sim-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
