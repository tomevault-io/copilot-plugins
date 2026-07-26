## go-redis

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Repository guidance — layout, commands, architecture, and conventions — is
maintained in [`AGENTS.md`](AGENTS.md) so it stays shared across AI tools. It is
imported below; edit `AGENTS.md`, not this file, for that content.

@AGENTS.md

## Claude-specific

- Repo-local **skills** under `.claude/skills/` auto-trigger from their
  descriptions — no need to invoke them manually (the set is listed in
  `AGENTS.md`).
- **Slash commands** under `.claude/commands/` (e.g. `/check-ci`) are available
  in-session.
- Architectural **specs** under `.claude/specs/` are read on demand; open the
  relevant one before changing that subsystem.

---
> Source: [redis/go-redis](https://github.com/redis/go-redis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
