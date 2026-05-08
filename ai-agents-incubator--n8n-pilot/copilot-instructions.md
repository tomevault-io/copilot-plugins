## n8n-pilot

> This file is the Codex entry orchestrator for projects that use the framework.

# Codex Adapter Entry

## Purpose

This file is the Codex entry orchestrator for projects that use the framework.
It runs Codex workflows on top of shared project memory.

## Shared State

Primary memory files:
- `.claude/SNAPSHOT.md`
- `.claude/BACKLOG.md`
- `.claude/ARCHITECTURE.md`
- `CHANGELOG.md` (optional, if project tracks release notes)

Codex and Claude use the same state contract.

## Command Routing

### `start`
Run:
- `bash .codex/commands/start.sh`

### `/fi`
Run:
- `bash .codex/commands/fi.sh`

### migration detection
Run:
- `bash .codex/commands/migration-router.sh`

### version check
Run:
- `bash .codex/commands/update-check.sh`

## Core Runtime

Shared command entry points:
- `python3 src/framework-core/main.py cold-start`
- `python3 src/framework-core/main.py completion`

Output contract:
- `.codex/contracts/core-cli-contract.md`

---
> Source: [AI-agents-incubator/n8n-pilot](https://github.com/AI-agents-incubator/n8n-pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
