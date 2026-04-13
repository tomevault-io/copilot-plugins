## rocinante

> Claude Code preferences for this repository. See [AGENTS.md](AGENTS.md) for full technical context.

# CLAUDE.md

Claude Code preferences for this repository. See [AGENTS.md](AGENTS.md) for full technical context.

## Git Workflow

All changes must go through a branch and pull request. Never commit directly to `main`.

## Model Usage Strategy

Optimize quota by using each model where it adds the most value:

- **Opus** (main conversation): Planning, architecture, code review, complex decisions
- **Sonnet** (subagents): Implementation, coding tasks, mechanical changes
- **Haiku**: Quick lookups, simple searches

When dispatching subagents for implementation work, use `model: "sonnet"`.
When dispatching subagents for code review, use `model: "opus"`.

## Running System

When helping on the deployed rocinante system, see [SKILL.md](SKILL.md) for runtime guidance (also deployed at `/usr/share/rocinante/SKILL.md` on the running system).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/allardvdb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/allardvdb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
