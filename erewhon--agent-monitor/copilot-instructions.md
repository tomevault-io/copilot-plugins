## agent-monitor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
# Build the binary
go build -o agent-monitor .

# Install all components
cp agent-monitor agent-monitor-session focus-agent-monitor ~/.local/bin/
cp tmux-outer.conf ~/.config/agent-monitor-tmux.conf

# Launch the full monitor session (outer tmux + TUI + preview pane)
agent-monitor-session

# Run TUI standalone (no outer tmux integration)
agent-monitor --no-attach

# List agents non-interactively (for scripts/status bars)
agent-monitor --list
```

## Architecture

Single-file Go TUI (`main.go`) using the [Bubble Tea](https://github.com/charmbracelet/bubbletea) framework (Elm architecture: Model/Update/View). No tests currently.

**Core types:**
- `Agent` — represents a coding agent instance (Claude Code, OpenCode, or Crush) detected in a tmux pane (session:window.pane targeting)
- `AgentType` — enum: Claude/OpenCode/Crush/Unknown, determines which detection patterns to use
- `Model` — Bubble Tea model holding agent list, cursor position, groups, flatAgents, and outer tmux socket reference
- `AgentStatus` — enum: Running/Waiting/Idle/Error, detected via pattern matching on captured tmux pane content
- `Config` / `GroupConfig` — YAML-mapped structs for agent grouping config
- `Group` — `{Name string, Agents []Agent}` for grouped rendering

**Config file:** `~/.config/agent-monitor/groups.yaml` — optional YAML file that defines named groups of agents by session name. If missing or malformed, agents display in a flat list.

**Agent detection flow:** `detectAgents()` calls `tmux list-panes -a` on the default socket, matches pane commands against `claude`/`opencode`/`crush`, then falls back to content probes (`looksLikeClaude`/`looksLikeCrush`/`looksLikeOpenCode`) for wrapped processes. `detectAgentStatus()` dispatches to type-specific detectors (`detectClaudeStatus`/`detectCrushStatus`/`detectOpenCodeStatus`) that match against each tool's UI patterns.

**Nested tmux design:** The monitor runs inside an "outer" tmux session on a separate socket (`-L agent-monitor`) with prefix `C-a`, so it doesn't conflict with users' regular tmux (`C-b`). The outer session has two panes: left = TUI, right = live attach to selected agent's tmux session via `respawn-pane`.

**Shell scripts:**
- `agent-monitor-session` — launcher that creates the outer tmux session with split layout
- `agent-monitor-placeholder` — placeholder display for the right pane before an agent is selected
- `focus-agent-monitor` — helper to return focus from inner tmux to the monitor pane

## Task Management

Task specs and feature specs live in the **Forge** notebook in Nous. When working on a task:

- Use `mcp__nous__get_page` to read the task spec from Forge (e.g., "Task: Kanban Board Web UI")
- Use `mcp__nous__get_database` on the "Project Tasks" database in Forge to see status and dependencies
- Update task status via `mcp__nous__update_database_rows` in the Project Tasks database (not internal task tools)
- Feature pages in Forge contain the full context: data model, API contracts, edge cases, test plans

Do NOT create ad-hoc task tracking internally — all task state lives in Forge.

## Version Control

This repo uses **jj** (Jujutsu), not git.

---
> Source: [erewhon/agent-monitor](https://github.com/erewhon/agent-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
