## alba

> Welcome! This is your ALBA agent. Run `/setup` to create your personalized AI agent system.

# ALBA

Welcome! This is your ALBA agent. Run `/setup` to create your personalized AI agent system.

## Quick Start

1. Open Claude Code in this directory
2. Run `/setup`
3. Answer 7 questions about your role, tools, and preferences
4. Your personalized agent system is ready

## What Gets Created

`/setup` builds a complete agent system tailored to you:

- **CLAUDE.md** - Your agent's brain (replaces this file)
- **memory/** - Persistent memory across sessions (tasks, learnings, errors)
- **skills/** - Custom capabilities (research, automation, etc.)
- **hooks/** - Automated workflows (session start, error logging, safety checks)
- **rules/** - Behavioral guidelines (decision protocol, security)

## After Setup

| Command | Purpose |
|---------|---------|
| `/start` | Begin session, load context |
| `/end` | End session, save state |
| `/status` | Quick overview |
| `/extend` | Add new features anytime |

## Documentation

| Resource | Location |
|----------|----------|
| Templates | `templates/INDEX.md` |
| Examples | `examples/` |

## Progressive Disclosure

After setup, your CLAUDE.md will use lazy-loading. Details stay in `.claude/docs/` and load only when needed - keeping your context window efficient.

---

*ALBA - Your routine-driven AI companion*
*Run `/setup` to begin.*

---
> Source: [onurpolat05/ALBA](https://github.com/onurpolat05/ALBA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
