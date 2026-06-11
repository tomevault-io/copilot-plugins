## session-handoff-skill

> Use this guide to install and run the **session-handoff** skill in Claude Code.

# Session Handoff — Claude Code

Use this guide to install and run the **session-handoff** skill in Claude Code.

## What it does

The skill writes a structured handoff file at `.cursor/state/HANDOFF.md` in your
project. That path is shared with Cursor so you can hand off work between clients
on the same repository.

It preserves explicit task state — role, objective, progress, decisions, blockers,
and next actions — not hidden model reasoning.

## Installation

Choose one method.

### Option A — Plugin (recommended)

Install from a local clone or marketplace entry.

**Local plugin directory:**

```bash
git clone <repo-url> ~/.claude/plugins/session-handoff-skill
claude --plugin-dir ~/.claude/plugins/session-handoff-skill
```

Or add the plugin through Claude Code's plugin/marketplace UI if you publish
`.claude-plugin/marketplace.json`.

The `.claude-plugin/plugin.json` file registers `./skills/`, exposing the
`session-handoff` skill.

### Option B — Project skills directory

Copy the skill into your repo:

```bash
mkdir -p .claude/skills
cp -R /path/to/session-handoff-skill/skills/session-handoff .claude/skills/session-handoff
```

Skills under `.claude/skills/` are discovered for that project.

### Option C — User skills directory

Install for all projects:

```bash
mkdir -p ~/.claude/skills
cp -R /path/to/session-handoff-skill/skills/session-handoff ~/.claude/skills/session-handoff
```

### Option D — Via Agent

In Claude Code:

```
Install the session-handoff skill from <repo-url> and confirm it is available.
```

## Verify installation

Start Claude Code in a project and ask:

```
List available skills related to session handoff.
```

Or invoke directly:

```
/session-handoff:session-handoff status — do not write files
```

## Usage

The plugin exposes the `session-handoff` skill, but the skill sets
`disable-model-invocation: true` — invoke it explicitly with the namespaced
slash command `/session-handoff:session-handoff` (plugin install) or by
referencing the skill when installed via `.claude/skills/`.

| Mode | When to use | Example prompt |
|------|-------------|----------------|
| **save** | End of session; before compacting context | `/session-handoff:session-handoff save current context` |
| **resume** | Continue previous work | `/session-handoff:session-handoff resume from HANDOFF.md` |
| **update** | After completing a milestone | `/session-handoff:session-handoff update the handoff file` |
| **status** | Read-only summary | `/session-handoff:session-handoff status — do not write files` |

### Save before you leave

```
/session-handoff:session-handoff save context. Include git status, completed
work, blockers, and the first executable next action.
```

Claude will create or replace `.cursor/state/HANDOFF.md`.

### Resume in a new session

```
/session-handoff:session-handoff resume. Read .cursor/state/HANDOFF.md, verify the repo
matches the handoff, report mismatches, then continue from Next Actions.
```

You can also paste the **Resume Prompt** block from the handoff file.

### Hand off from Cursor to Claude Code

1. In Cursor: `/session-handoff save context`
2. Commit or keep `.cursor/state/HANDOFF.md` in the repo.
3. In Claude Code: `/session-handoff:session-handoff resume`

### Hand off from Claude Code to Cursor

1. In Claude Code: `/session-handoff:session-handoff save context`
2. In Cursor: `/session-handoff resume`

## Handoff file location

| File | Purpose |
|------|---------|
| `.cursor/state/HANDOFF.md` | Canonical handoff (shared with Cursor) |

Create the directory if needed:

```bash
mkdir -p .cursor/state
```

**Git**: Commit the handoff for team continuity, or add to `.gitignore` for
local-only notes.

## Tips for Claude Code

- **Long sessions**: Save before `/compact` or when switching to a different model.
- **Subagents**: Include subagent results in `Completed Work` when updating.
- **Verification on resume**: The skill requires comparing `git status` and file
  existence before making new edits — do not skip this step.
- **No secrets**: API keys, tokens, and private keys must never appear in the handoff.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Skill not found | Confirm plugin path or `.claude/skills/session-handoff/SKILL.md` exists |
| Wrong handoff path | Use `.cursor/state/HANDOFF.md` for cross-client compatibility |
| Stale state after resume | Run update mode after verifying what changed |
| Permission errors | Ensure `.cursor/state/` is writable |

## See also

- [README.md](README.md) — overview and installation for all clients
- [EXAMPLES.md](EXAMPLES.md) — sample prompts and handoff snippets
- [CURSOR.md](CURSOR.md) — Cursor installation

---
> Source: [live9369/session-handoff-skill](https://github.com/live9369/session-handoff-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
