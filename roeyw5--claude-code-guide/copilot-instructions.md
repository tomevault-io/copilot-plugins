## claude-code-guide

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A shareable collection of Claude Code skills, hooks, and educational examples. Everything lives under `.claude/` so skills and hooks work immediately when you clone the repo — no copying required.

## Repository Structure

- `README.md` — Comprehensive guide to Claude Code fundamentals (memory, context, plan mode, skills, sub-agents, hooks)
- `.claude/skills/` — Reusable skills in `<name>/SKILL.md` format, usable as `/slash-commands`
- `.claude/agents/` — Example specialized sub-agents (e.g., `pr-reviewer`)
- `.claude/hooks/` — Security hooks (Node.js) for blocking dangerous commands and protecting secrets
- `examples/CLAUDE.md` — Sample global CLAUDE.md (the one in `~/.claude/CLAUDE.md`) provided as a starting point readers can copy and adapt
- `images/` — Diagrams referenced by the README

## Key Conventions

- Skills follow the `<name>/SKILL.md` directory convention with YAML frontmatter and optional `$ARGUMENTS` support
- Related skills are grouped with a shared prefix (e.g., `review-*`, `gen-*`)
- Hook scripts follow the PreToolUse pattern, log to `~/.claude/hooks-logs/`, and use configurable severity levels
- All example content uses generic placeholders — no real names or company-specific references

## Working With This Repo

No build or test commands. Changes are typically documentation edits, new skill files, or hook scripts. When adding new skills, create a `<name>/SKILL.md` folder under `.claude/skills/`. When adding hooks, add `.js` files under `.claude/hooks/`.

---
> Source: [roeyw5/claude-code-guide](https://github.com/roeyw5/claude-code-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
