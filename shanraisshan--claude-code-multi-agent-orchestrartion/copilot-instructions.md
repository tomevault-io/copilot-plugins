## claude-code-multi-agent-orchestrartion

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Does

A Claude Code multi-agent orchestration demo that fetches current temperatures from 195 countries' capitals in parallel, writes results to a file, then calculates the global average. Run it with `/orchestrate`.

## Architecture

The system uses three layers of Claude Code custom agents orchestrated by a slash command:

1. **`/orchestrate` command** (`.claude/commands/orchestrate.md`) — the entry point. Launches all agents in the correct order with dependency management.

2. **195 Weather Fetch Agents** (`.claude/agents/weather-fetch/agent-weather-{country}.md`) — one per country, each calls a country-specific MCP tool (`mcp__weather-mcp-shayan-http__get_{country}_weather_shayan`) and returns only `[number]°C`. All run in parallel via background tasks.

3. **Weather Writer Agent** (`.claude/agents/weather-writer/agent-weather-writer.md`) — receives all temperatures, writes `output/temperatures.md` in `Country = [temp]` format. Runs after all fetch agents complete.

4. **Weather Average Agent** (`.claude/agents/weather-average/agent-weather-average.md`) — reads `output/temperatures.md`, calculates the average, writes `output/average.md`. Runs after the writer finishes.

### Execution Flow

```
/orchestrate
  → 195 fetch agents (parallel, background)
  → wait for all
  → writer agent (writes output/temperatures.md)
  → average agent (reads temperatures, writes output/average.md)
```

## MCP Server

Weather data comes from a remote HTTP MCP server configured in `.mcp.json`:
- URL: `https://mcp-weather-j5kl.onrender.com/mcp`
- Provides 195 per-country tools named `get_{country}_weather_shayan`

## Hooks System

All hooks route through a single Python script: `.claude/hooks/scripts/hooks.py`. It plays sound effects for different Claude Code lifecycle events (tool use, session start/end, subagent activity, etc.).

- Sound files live in `.claude/hooks/sounds/{event}/`
- Per-hook enable/disable config: `.claude/hooks/config/hooks-config.json`
- Local overrides (git-ignored): `.claude/hooks/config/hooks-config.local.json`
- Agent-specific sounds use `agent_` prefixed folders (e.g., `agent_pretooluse/`)
- Logs written to `.claude/hooks/logs/hooks-log.jsonl` (unless `disableLogging: true`)

## Output Files

- `output/temperatures.md` — one line per country: `Country = X°C`
- `output/average.md` — single line: `Average = X°C`

## Git Commit Rules

When committing changes, **create separate commits per file**. Do NOT bundle multiple file changes into a single commit. Each file gets its own commit with a descriptive message specific to that file's changes.

For example, if `README.md`, `best-practice/claude-subagents.md`, and a skill file all changed:
- Commit 1: `git add README.md` → commit with README-specific message
- Commit 2: `git add best-practice/claude-subagents.md` → commit with subagents-doc-specific message
- Commit 3: `git add .claude/skills/weather-fetcher/SKILL.md` → commit with skill-specific message

This makes the git history cleaner and easier to review, revert, or cherry-pick individual changes.

## Adding a New Country Agent

Copy any existing agent file in `.claude/agents/weather-fetch/`, update the country name, capital, and MCP tool name. Then add the agent to `.claude/commands/orchestrate.md`.

---
> Source: [shanraisshan/claude-code-multi-agent-orchestrartion](https://github.com/shanraisshan/claude-code-multi-agent-orchestrartion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
