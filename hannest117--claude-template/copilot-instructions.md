## claude-template

> <!-- CLAUDE_SETUP Customize this section for your project -->

# CLAUDE.md - Project Instructions for Claude Code

## Project Overview

<!-- CLAUDE_SETUP Customize this section for your project -->

## Development Workflow

1. Make changes
2. Run typecheck
3. Run tests
4. Lint before committing
5. Before creating PR: run full lint and test suite

## Code Style & Conventions

## Commands Reference

See @../docs/commands.md

## Self-Improvement

After every correction or mistake, update this CLAUDE.md with a rule to prevent repeating it. 

## Working with Plan Mode

- Start every complex task in plan mode (shift+tab to cycle)
- Pour energy into the plan so Claude can 1-shot the implementation
- When something goes sideways, switch back to plan mode and re-plan. Don't keep pushing.
- Use plan mode for verification steps too, not just for the build

## Parallel Work

- For tasks that need more compute, use subagents to work in parallel
- Offload individual tasks to subagents to keep the main context window clean and focused
- When working in parallel, only one agent should edit a given file at a time
- For fully parallel workstreams, use git worktrees:
  `git worktree add .claude/worktrees/<name> origin/main`

## Things Claude Should NOT Do

<!-- CLAUDE-SETUP Add mistakes Claude makes so it learns -->

- Don't use `any` type in TypeScript without explicit approval
- Don't skip error handling
- Don't commit without running tests first
- Don't make breaking API changes without discussion

## Favoured patterns

See @../docs/conventions/patterns.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HannesT117)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HannesT117)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
