## iterative-memory

> Update memory files as work happens — not at session end. Read on every action.


# Iterative Memory — Always-On Rule

Memory upkeep is part of the work, not bookkeeping after.

**Before responding, ask yourself:** did anything just happen that should land in `01-context/state.md`, `activity-log.md`, `decisions-log.md`, or `insights.md`? If yes, do that update **before** continuing.

**Trigger → file:**
- Decision made (closed or opened) → `decisions-log.md`
- Finding, quirk, gotcha, or pattern → `insights.md`
- Step completed (commit, deploy, gate passed, milestone met) → `activity-log.md` + `state.md`
- External system state changed (deploy, secret, schedule, data) → `state.md`
- Pausing for human review → `state.md` (Awaiting Human Decision)
- Context-window pressure / compaction imminent → full `state.md` flush first
- User says "checkpoint" or runs `/checkpoint` → full pass + commit

**Commit after each checkpoint, not in batches.** Logged-but-uncommitted state doesn't survive thread switches.

**Say it out loud** when you update a memory file ("logged decision X," "refreshed state.md"). The user needs to see the discipline working — or notice when it isn't.

**At session start, read:** `CLAUDE.md` → `01-context/state.md` → last 5 `activity-log.md` entries → open items in `decisions-log.md`. If `state.md` is current, chat history shouldn't matter for resumption.

---
> Source: [16wells/divi-docs](https://github.com/16wells/divi-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
