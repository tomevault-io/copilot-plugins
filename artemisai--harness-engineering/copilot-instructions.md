## harness

> Harness Engineering — rules for all files


Read `AGENTS.md` for the full project spec.

## Session rules

- Implement exactly one feature per session from `features.json`
- Run `bash init.sh` first, then read `claude-progress.txt` and `features.json`
- End every session with an updated `claude-progress.txt` entry and committed code

## Code rules

- Commits: `feat(<id>): ...` / `fix(<id>): ...` / `chore(entropy): ...`
- No WIP on `main`. Every commit leaves the codebase buildable.
- Layer order: `Types → Config → Repo → Service → Runtime → UI`

## Skills

Reference with `@skill-name`. Available skills in `skills/*.skill.md`:
`@session-start`, `@session-end`, `@implement-feature`, `@new-feature`, `@run-tests`,
`@create-issue`, `@delegate-subagent`, `@entropy-check`, `@write-adr`, `@assign-agent`

## Never

`rm`, `del`, `rmdir`, `kill`, `sudo`, `&&`, `||`, path traversal `..`

---
> Source: [ArtemisAI/Harness_Engineering](https://github.com/ArtemisAI/Harness_Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
