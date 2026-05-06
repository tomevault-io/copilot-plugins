## bug-tracking

> Bug tracking and logging rules


# Bug Tracking

Every bug discovered during development or testing MUST be logged in `docs/bug-log.md`.

## When to log

- Any time a runtime error, incorrect behavior, or UX issue is found during manual testing
- Any time a user reports unexpected behavior
- Any time a fix is applied for a bug (update status to "Fixed")

## Entry format

Each entry must include:
- **BUG-XXX** sequential ID
- **Found during:** which test scenario or flow
- **Severity:** Critical (blocks flow), High (confusing/broken), Medium (workaround exists), Low (polish)
- **Status:** Fixed / Open / Known limitation
- **Symptom:** what the user sees
- **Root cause:** why it happens (be specific about the code path)
- **Fix:** what was changed
- **Files changed:** list of affected file paths

## Rules

1. Never delete bug entries — mark them as "Fixed" instead
2. When fixing a bug, update the entry with the fix description and changed files
3. Use the template at the bottom of `docs/bug-log.md` for new entries
4. Increment the BUG-XXX number sequentially
5. When multiple bugs are found in one session, log them all before moving on

---
> Source: [HiLuLiT/pet-circle](https://github.com/HiLuLiT/pet-circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
