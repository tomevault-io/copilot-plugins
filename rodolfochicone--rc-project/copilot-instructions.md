## rc-project

> Guidance for coding agents working in **this repository**. RC is a **pure agent plugin** — skills,

# AGENTS.md

Guidance for coding agents working in **this repository**. RC is a **pure agent plugin** — skills,
agents and hooks, plus a few small Node/Bash helpers. No compiled binary, no build step, no CLI.

## Project overview

RC drives AI-assisted development — spec → tickets → implement → review → ship — through components
the agent host loads. It ships as a Claude Code plugin, and nothing in the shipped set is coupled to a
host-exclusive tool; all runtime state is plain files under a project's `.rc/`.

Three primitives, one job each: **hooks** hold rules that must apply every time, **skills** hold the
disciplines the agent reaches for, **agents** exist for context isolation. A component in the wrong
primitive is the most common defect in this repo — a sequence of skills is not a skill, and an
every-time rule written as prose is not enforced.

## Repository layout

| Path | Contents |
| --- | --- |
| `skills/promoted/<name>/SKILL.md` | The 17 shipped skills. Each: frontmatter + optional `references/`, `assets/`, `scripts/`. |
| `skills/stacks/<name>/` | Stack skills, **not** shipped — loaded by name when working that stack. |
| `skills/misc/<name>/` | Kept on disk, **not** shipped. Zero context cost. |
| `agents/<name>.md` | The four council archetypes, dispatched by `rc-council`. |
| `commands/<name>.md` | Two thin prompt macros with no skill behind them. |
| `hooks/hooks.json` + `hooks/scripts/*.sh` | Lifecycle hooks (bash), gated via `_lib.sh`. |
| `scripts/*.mjs` | `plugin-smoke.mjs` (the gate), `validate-tasks.mjs`, `lessons.mjs`. |
| `docs/` | Human-facing docs. |
| `.claude-plugin/` | `plugin.json` + `marketplace.json`. |

## The budget that shapes everything

Every shipped description loads in **every** session, against the host's skill-listing budget
(`skillListingBudgetFraction`, 1% of context by default). Over budget, the host **drops descriptions**
and the skill goes mute — present by name, nothing to match against. That is why:

- Only `skills/promoted/` is in `plugin.json`. Adding to `stacks/` or `misc/` is free; **promoting is
  the deliberate trade.**
- Promoted descriptions are capped at **150 characters**, trigger plus anti-trigger.
- `PROMOTED_CEILING` in `plugin-smoke.mjs` is **17**. To cross it: merge one, demote one, or raise the
  constant on purpose.

## Developing this repo

There is no build. Editing components is editing markdown, JSON and small scripts.

```bash
node scripts/plugin-smoke.mjs                 # the gate — must pass
node scripts/validate-tasks.mjs --selftest
node scripts/lessons.mjs --selftest
bash hooks/scripts/<hook>.sh --selftest
```

`plugin-smoke.mjs` must pass before a change is complete. It checks frontmatter, that `name` matches
the directory, the bucket/manifest split in both directions, description length, anti-triggers, README
catalog presence, dangling references, orphan bundled files, hook wiring in both directions, any
dependency on the Claude-only `Workflow` tool, the promoted ceiling, and that
`docs/manual/rc-manual.html` still matches the components the repo ships.

## Git — hands-off by default

- **NEVER** run `git restore`, `git checkout`, `git reset`, `git clean`, `git rm`, commit, push or
  branch **without explicit user permission** — these can permanently lose work.
- If the worktree holds unexpected edits, read them and work around them; do not revert them.

## Conventions

- **Skills** — frontmatter `name` (matching the directory) and `description` (required), plus `model`,
  `effort`, `user-invocable`, `argument-hint` as needed. The description is a routing rule, not a
  summary: front-load the trigger, include a "Do not use for…" clause, stay under 150 characters when
  promoted. Deep content goes in `references/`, never inlined.
- **Shared rules live once.** `.rc` resolution, Serena navigation, untrusted content, root cause over
  workaround, the verification-evidence bar and project memory are in
  `skills/promoted/rc/references/conventions.md`. A skill points at it; it never re-states it. Those
  blocks were pasted into 8 skills before, and a fix landed in one copy and rotted in the others.
- **Agents** — frontmatter `name`, `description` (required), `tools` (least privilege), `model`,
  `color`. Council archetypes are leaf workers: no `Task`/`Agent` tool, so they cannot recurse.
- **Hooks** — bash, `set -u`, source `_lib.sh`, gate with `rc_hook_active <name> <profile>`, **fail
  open on any environment problem**, and ship a `--selftest`. Prefer `rc_ask` over `rc_deny` when the
  pattern has a legitimate form; a guard that blocks legitimate work gets disabled. Reference scripts
  with `${CLAUDE_PLUGIN_ROOT}`.
- **Commands** — only for a prompt macro with no skill behind it. "Invoke skill A, then B, then C" is a
  workflow in prose; that belongs inside a skill.
- **Docs in sync** — when a skill's behavior, trigger or output changes, update its description, its
  `references/`, `skills/promoted/rc/SKILL.md` and the README catalog. If no doc change is needed, say so.
- **Match the surrounding style.** Read neighbouring components before adding one.

## Skill dispatch when using RC on a project

Prefer these for their domain, and activate every skill whose domain the change touches:
`rc-diagnose` (any bug, failure, or tracing question — root cause before any fix), `rc-tdd` (writing
or changing tests), `rc-code-review` (before a PR), `rc-memory` (before working, and after a
correction), `rc-writing-for-agents` (authoring or trimming anything an agent reads).

## Anti-patterns

1. Marking a change done without `node scripts/plugin-smoke.mjs` passing.
2. Editing a component and leaving `skills/promoted/rc/SKILL.md` or the README catalog stale.
3. **Promoting a skill without demoting one** — that is how 48 skills happened, and how they went mute.
4. Re-stating a shared rule inside a skill instead of pointing at `conventions.md`.
5. Writing an every-time rule as prose when it could be a hook.
6. Building on the `Workflow` tool — Claude-Code-only, and the gate rejects it.
7. Referencing the retired `rc` CLI (`rc setup`, `rc exec`, `rc tasks run`).
8. Running destructive git commands without explicit permission.
9. Fixing a bug by patching the symptom instead of the root cause.

---
> Source: [rodolfochicone/rc-project](https://github.com/rodolfochicone/rc-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
