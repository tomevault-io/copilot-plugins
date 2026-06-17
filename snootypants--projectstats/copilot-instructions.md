## projectstats

> This file coordinates multi-agent swarm mode for ProjectStats.

# SKILL.md — ProjectStats Agent Teams (Swarm) Configuration

## Purpose
This file coordinates multi-agent swarm mode for ProjectStats.
When swarm mode is active, multiple Claude Code agents can work in parallel
on different parts of the codebase.

## Coordination Rules

### File Ownership
- Each agent claims files it is actively editing
- No two agents should edit the same file simultaneously
- Agents must check this file before modifying shared resources

### Communication Protocol
- Agents report progress by updating their section below
- Blocked agents should note what they're waiting for
- Completed agents mark their section as DONE

### Conflict Prevention
- Always pull latest before starting work
- Use feature branches when possible
- Coordinate through commit messages with [SWARM] prefix

## Active Agents
<!-- Agents register here when working -->

| Agent | Task | Files | Status |
|-------|------|-------|--------|
| — | — | — | Idle |

## Shared State Warnings
- Database migrations must be sequential
- Package.swift / package.json changes need coordination
- CI/CD config changes require team review

---
> Source: [Snootypants/projectStats](https://github.com/Snootypants/projectStats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
