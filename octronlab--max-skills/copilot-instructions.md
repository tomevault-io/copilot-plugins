## max-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal skills marketplace: Markdown skills plus JSON manifests. No build, no tests, no dependencies.

Two checks, one per distribution channel. Run both after changing a skill folder or either file in `.claude-plugin/`:

```bash
claude plugin validate .                    # plugin manifests
npx skills@latest add . --list              # skills.sh discovery
```

The skills CLI has no `validate` subcommand; `add --list` is the substitute, and it fails loudly on a malformed `SKILL.md` frontmatter. To confirm what an install actually copies, install into a throwaway git repo with `-s '*' -a claude-code -y --copy` and inspect `.claude/skills/`.

To exercise a change end to end, add the working copy as a local marketplace so edits are live without a push:

```
/plugin marketplace add /Users/mamnun/workspace/me.mamnun/max-skills
/plugin install max-skills@max
```

Skills and output styles load at session start, so a restart or `/clear` is needed before a change is visible.

Installed copies do not auto-update. After pushing, consumers pull with `/plugin marketplace update max` or `npx skills@latest update`, so bump `version` in `plugin.json` on any change worth pulling.

## Dual distribution, and the constraint it imposes

The repo ships through two channels that package it differently:

- **Claude Code plugin**: `.claude-plugin/marketplace.json` (marketplace `max`) and `.claude-plugin/plugin.json` (plugin `max-skills`, which lists every skill path explicitly).
- **skills.sh** (`npx skills@latest add OctronLab/max-skills`): the Codex-compatible route. It copies **skill folders** and nothing else.

That second channel is the architectural constraint: **anything a skill needs at runtime must live inside that skill's own folder.** A top-level asset directory reaches plugin users and silently vanishes for skills.sh users. This is why the output styles live at `skills/setup-max-skills/output-styles/` rather than at the repo root, even though the plugin format supports a root `output-styles/`.

A new skill therefore needs three things, not one:

1. `skills/<name>/SKILL.md`
2. `skills/<name>/agents/openai.yaml` (Codex metadata; `policy.allow_implicit_invocation: false` is Codex's equivalent of `disable-model-invocation: true`)
3. Its path added to the `skills` array in `plugin.json`

## Two frontmatter contracts

`SKILL.md` and an output-style `.md` look alike and are not interchangeable.

Skills use `name`, `description`, `disable-model-invocation`, `argument-hint`. Output styles use `name`, `description`, `keep-coding-instructions`. An output style with `keep-coding-instructions: true` layers prose rules over Claude Code's engineering instructions; without it, those instructions are dropped entirely.

## Conventions

**Skills are prompt-driven, not scripts.** They tell the agent to explore, present a diff, confirm, then write. Preserve that shape rather than replacing it with commands to execute blindly.

**Skills that change machine state must converge.** `setup-max-skills` is run repeatedly, so it diffs before writing, treats an already-correct value as a no-op, and never reverts a choice the user made later. Any new stateful skill needs the same property.

**Output styles are Claude Code only.** Codex has no equivalent, so `setup-max-skills` exits early on any non-Claude-Code agent instead of writing settings nothing reads.

**`output-styles/i-have-adhd.md` is vendored** from [ayghri/i-have-adhd](https://github.com/ayghri/i-have-adhd) (MIT). Its body below the frontmatter is upstream's; the frontmatter is local. Step 5 of `setup-max-skills` syncs it. Don't edit the body except to pull an upstream change.

## Git

`origin` is SSH (`git@github.com:OctronLab/max-skills.git`). HTTPS pushes fail here: something outside git config authenticates as a stale `max-healthengine` identity and gets a 403. Keep the remote on SSH.

---
> Source: [OctronLab/max-skills](https://github.com/OctronLab/max-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
