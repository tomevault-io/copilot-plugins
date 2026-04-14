## harness-engineering

> Read `AGENTS.md` at the project root first. This file extends it for Windsurf / Cascade.

# Harness Engineering — Windsurf Rules

Read `AGENTS.md` at the project root first. This file extends it for Windsurf / Cascade.

## Session protocol
1. `bash init.sh`
2. Read last 30 lines of `claude-progress.txt` and parse `features.json`
3. Check `agents.json` — register session, claim highest-priority failing feature
4. Implement exactly one feature per session

## Skills
Reference with `@skill-name` in Cascade chat:
`@session-start` `@session-end` `@implement-feature` `@new-feature` `@run-tests`
`@create-issue` `@delegate-subagent` `@entropy-check` `@write-adr`

## Rules
- No WIP on `main`. One feature per session. No scope expansion.
- Layer order: `Types → Config → Repo → Service → Runtime → UI`
- Blocked: `rm`, `del`, `rmdir`, `kill`, `sudo`, `&&`, `||`
- Update `claude-progress.txt` and `agents.json` before ending session

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArtemisAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
