## compabob

> Claude Code loads this file every session. It imports the constitution and points at memory.

# Compabob

Claude Code loads this file every session. It imports the constitution and points at memory.

@CONSTITUTION.md

## Memory

At the start of a session, read `memory/MEMORY.md`. It records the user's name, role, and working language, your own name, and the index of what you have learned about the user and their work. Follow its links into `memory/topics/` only as needed.

## Working directory

The knowledge base is `vault/`. Persistent facts are in `memory/`. Your own configuration is in `.claude/` and `config/`. The directories `vault/`, `memory/`, and `config/` are the user's own data and live outside the kit's version control. Everything else is the kit, and updates upstream.

---
> Source: [chacosoldier/compabob](https://github.com/chacosoldier/compabob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
