## bonfire

> Guidance for AI coding agents working with this repository.

# AGENTS.md

Guidance for AI coding agents working with this repository.

## Overview

Bonfire provides session context persistence for AI coding. Pick up exactly where you left off.

**Installation:**
```bash
npx skills add vieko/bonfire
```

## Commands

| Command | Outcome |
|---------|---------|
| `/bonfire start` | Session started, context loaded, ready to work |
| `/bonfire end` | Work captured, context healthy |

## Skill Structure

```
skills/bonfire/
├── SKILL.md              # Command routing
├── commands/             # Outcome definitions
│   ├── start.md
│   └── end.md
└── templates/            # Default files
    ├── index.md          # Session context template
    └── gitignore.md      # Git ignore template
```

## Design Principles

Commands define **outcomes, not procedures**:
- What success looks like
- How to verify it worked
- Boundaries and constraints

The agent determines the procedure.

## Session Context

This repo uses bonfire. Context is in `.bonfire/index.md`.

## Links

- [skills.sh](https://skills.sh) - Installation
- [agentskills.io](https://agentskills.io) - Specification

---
> Source: [vieko/bonfire](https://github.com/vieko/bonfire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
