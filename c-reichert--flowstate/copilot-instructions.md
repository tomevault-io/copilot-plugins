## flowstate

> A Claude Code plugin providing structured thinking workflows: brainstorming, planning, TDD, compound (incremental documentation), and multi-agent review.

# FlowState — Claude Code Plugin

## What This Is
A Claude Code plugin providing structured thinking workflows: brainstorming, planning, TDD, compound (incremental documentation), and multi-agent review.

## Architecture
- `/skills/` — each subdirectory is a skill (SKILL.md defines behavior)
- `/agents/` — agent definitions for research and review subagents (`research/` + `review/`)
- `/commands/workflow/` — workflow commands (brainstorm, write-plan, work, review, compound, etc.)
- `/hooks/` — session-start hook that injects core rules
- `/.claude-plugin/plugin.json` — plugin manifest + version (source of truth)
- `/.claude-plugin/marketplace.json` — marketplace manifest, must mirror plugin.json version

## Conventions
- Version lives in `.claude-plugin/plugin.json` — bump on every feature/fix commit
- Bumping version: edit `version` in `plugin.json` AND `marketplace.json` (both must match), commit as `chore: bump version to X.Y.Z`
- Skill bodies use **markdown**, not XML tags. Use `>` blockquotes for emphasis blocks. (Earlier guidance recommended XML — superseded by Anthropic's Opus 4.6 prompting research, which prefers markdown structure.)
- The `write-plan` skill exists because `plan` conflicts with Claude's built-in EnterPlanMode
- All commands set `disable-model-invocation: false` so Claude can dispatch workflows when contextually appropriate. The skill bodies themselves contain the user-approval gates.

## Defend Research Before Reverting
When you have completed research that supports a specific recommendation, and the user immediately overrides it without engaging with the evidence, present the key evidence before making changes. A single sentence like "My research found X — would you like to see the evidence before I revert?" prevents costly flip-flops where work gets done, undone, and redone. This applies to any researched recommendation, not just prompting conventions.

---
> Source: [c-reichert/flowstate](https://github.com/c-reichert/flowstate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
