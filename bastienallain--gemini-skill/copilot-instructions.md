## gemini-skill

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code skill/plugin** that provides "second opinion" functionality by integrating Google's Gemini CLI. When Claude Code creates a plan, Gemini automatically reviews it via a PostToolUse hook before the user approves.

## Architecture

```
.claude-plugin/plugin.json   - Plugin metadata (name, version, skill paths)
skills/gemini/SKILL.md       - Skill definition (invoked via /gemini or hooks)
hooks/plan-review.sh         - Bash hook triggered on ExitPlanMode
hooks/settings-snippet.json  - Template for ~/.claude/settings.json hook config
```

- **No build system, no dependencies, no tests** — this is a configuration-only package
- Skills are Markdown files with YAML frontmatter (`name`, `description`)
- The hook reads plan content from stdin (JSON with `tool_input.planFile` and `cwd`), calls `gemini -p`, and outputs the review to stdout

## Key CLI Command

The core integration point is:
```bash
gemini -p "prompt here"
```
This is Gemini CLI's headless/non-interactive mode. Model selection uses `--model <name>`.

## Installation Flow

Users install by copying `skills/gemini/SKILL.md` to `~/.claude/skills/gemini/` and `hooks/plan-review.sh` to `~/.claude/hooks/`, then adding the hook config from `settings-snippet.json` to their `~/.claude/settings.json`.

## Models

- `gemini-3-flash-preview` — default, fast queries
- `gemini-3-pro-preview` — deep analysis, architecture decisions

---
> Source: [bastienallain/gemini-skill](https://github.com/bastienallain/gemini-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
