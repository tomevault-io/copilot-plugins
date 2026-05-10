## plant-monitor

> After completing a task from `plans/high-level-plan.md`, update its status row

# Claude Code Directives for plant-monitor

## Plan tracking

After completing a task from `plans/high-level-plan.md`, update its status row
from `❌` to `✅` in that file and include the plan update in the same commit as
the implementation work.

## Before touching dependency or toolchain versions

Do NOT downgrade `go` directives, module versions, or any other dependency
versions without first confirming with the user. If a build fails due to a
toolchain mismatch or missing network access, investigate the root cause and
ask before changing any version pins.

## Commit discipline

- One logical change per commit.
- Commit message format: `type(scope): short description` (conventional commits).
- Always push to the designated `claude/...` branch; never push to main/master
  without explicit permission.

## Branch naming

Development branches follow the pattern `claude/<description>-<sessionId>`.
Always verify the target branch before committing.

## Pull request descriptions

When opening or updating a PR, write a description that:
- Starts with a short (1–2 sentence) **summary** of the problem being solved and
  the approach taken — not a list of files changed.
- Has a **Changes** section with concise bullet points, one per logical change,
  explaining the *why* not just the *what*.
- Has a **Testing** section noting how the change was verified (unit tests,
  manual smoke test with specific env vars, etc.).
- Stays under ~250 words total. Be direct; omit filler like "this PR updates…".

---
> Source: [quentinrf/plant-monitor](https://github.com/quentinrf/plant-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
