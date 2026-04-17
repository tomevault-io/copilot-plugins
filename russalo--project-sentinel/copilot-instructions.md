## project-sentinel

> This file contains standing instructions for AI agents (Claude Code and others) working

# Agent Instructions — Project Sentinel

This file contains standing instructions for AI agents (Claude Code and others) working
in this repository. Read this before planning or implementing anything.

---

## Backlog Maintenance

The file `docs/BACKLOG.md` is the single source of truth for work that was discovered,
deferred, or left incomplete. You are required to keep it current.

**During a coding session, append to `docs/BACKLOG.md` when you:**
- Discover a bug, inconsistency, or technical debt that is out of scope for the current task
- Identify something that should be done soon but was not part of your planning
- Leave a task incomplete because it requires more information or a separate planning session
- Notice that documentation is out of sync with the current state of the code

**At the end of a session, remove from `docs/BACKLOG.md` when:**
- An item was fully resolved during the session
- An item is no longer relevant due to a direction change

**Format for new entries:**
```
- [ ] Short description of the item
      _Discovered: YYYY-MM-DD | Context: brief note on where/why this surfaced_
```

Add items under the most appropriate existing section. If no section fits, add one.
Never leave `docs/BACKLOG.md` in a state where completed work is still listed as pending.

---

## Explicit Approval Required Before Implementation

**You must not create, edit, or delete any file until the user has explicitly approved
a plan in the current session.**

This rule has no exceptions. "The plan seemed clear" is not an exception. "The user
approved something similar before" is not an exception. "The task is small" is not
an exception.

**Allowed without approval:**
- Reading files, exploring the codebase, running read-only commands
- Writing or updating the plan file at `/root/.claude/plans/`
- Asking clarifying questions

**Not allowed until the user says to proceed:**
- Creating new files
- Editing existing files
- Deleting files
- Running commands that modify state (git commits, installs, etc.)

If you have written a plan and are ready to implement, present it and stop. Wait for
the user to explicitly say to proceed — words like "go ahead", "do it", "looks good",
or equivalent. An acknowledgment that you answered a question correctly ("ok", "that
makes sense") is **not** approval to implement.

If you catch yourself about to write a file without approval, stop and ask.

---



You are expected to think critically, not just execute. Before implementing:

- If a directive is ambiguous, ask a clarifying question rather than assuming
- If you believe the approach has a meaningful tradeoff, name it explicitly
- If a requested tool, pattern, or dependency introduces lock-in or complexity that
  may not be worth the benefit, say so with your reasoning
- If something you were asked to do turns out to be more complex than expected,
  stop, surface the complexity, and propose that it go to the backlog rather than
  delivering a partial or risky implementation

Healthy critique is part of the job. Silent compliance that ships a wrong answer is not.

---

## Development Branch

All work goes to `claude/setup-cloud-environment-VGYBM` unless explicitly directed otherwise.
Never push to `main` directly.

---

## Directory Conventions

- `docs/` — project documentation and working reference files (BACKLOG.md lives here)
- `scripts/` — shell scripts for automation and dev lifecycle
- `mcp-servers/` — Python MCP server implementations
- `infrastructure/` — Docker Compose and environment configuration
- `tests/` — test suites (pytest for Python, pnpm test for TypeScript)
- `artifacts/` — deployable application packages (structure under review; see BACKLOG)
- `lib/` — shared libraries (structure under review; see BACKLOG)

---

## Things to Know About This Project

- This is a cross-OS project. Do not write scripts or configs that assume linux-only.
- Replit was the original development platform. We are migrating away from it.
  Do not introduce new `@replit/*` dependencies. See `docs/BACKLOG.md` for the
  full audit and removal plan.
- The frontend strategy for 1.0 is undecided. Do not build new frontend features
  without explicit direction.
- `just` is the command runner. Add new recipes to `justfile` rather than creating
  standalone scripts unless the logic is complex enough to warrant a separate file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/russalo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
