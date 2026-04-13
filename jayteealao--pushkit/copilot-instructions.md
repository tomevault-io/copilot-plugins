## pushkit

> Start: say hi. One motivating line. Then work.

# CLAUDE.md — generic (strict, clear, production)

Start: say hi. One motivating line. Then work.

## Owner / contact
- Owner: <FILL ME> (name, handle, email).
- Escalate on: prod incidents, security issues, secrets, signing/release keys, CI failures that block shipping.

## Style goals (always)
- Simple, clear, readable. Production-grade.
- Prefer explicit over clever. Readability counts.
- Small functions, clear names, clear data flow.
- Delete dead code, remove duplication, keep one source of truth.

## Non-negotiables (implementation)
- One canonical implementation in the primary codepath.
  - Remove legacy/shims/adapters in the same change.
  - No compatibility wrappers or transitional glue.
- Single source of truth for:
  - business rules, validation, enums, flags, constants, configuration.
- Thin clients:
  - UI/presentation layers render domain/shared models and options.
  - No business rules duplicated in UI.
- Validate and sanitize all user-controlled inputs before OS/file/process calls.
- Errors are explicit:
  - no silent catches.
  - add context when propagating.
  - logs have context, no secrets.

## Workflow
- No git worktrees unless user asks.
  - If asked: `peakypanes-worktress/<worktree-name>/`
- Safe git by default:
  - OK: `git status`, `git diff`, `git log`, `git show`.
  - No destructive ops unless explicitly requested (`reset --hard`, `clean`, `restore`, `rm`, …).
  - No amend unless asked.
- Small commits. Reviewable diffs. No repo-wide reformat.

## Process (how to work)
- Read relevant docs first (repo docs, specs, ADRs, CI workflows).
- Understand current architecture before changing it.
- Fix root cause, not symptoms.
- If stuck: capture exact error, minimal reproduction, propose 2–3 options with tradeoffs.

## Dependencies
- Avoid new dependencies.
- If required:
  - choose maintained, widely used options.
  - explain why, alternatives considered, and long-term cost.
  - remove anything replaced in the same change.

## Testing
- Behavior change => test change.
- Prefer deterministic tests:
  - no sleeps, no randomness, no real wall clock.
- Use the right level:
  - unit tests for pure logic.
  - integration/e2e tests for boundary behavior (DB, network, UI).
- Flaky tests get fixed or quarantined with a clear plan to unquarantine.

## Quality gates (run what the repo uses)
- Prefer repo task runner first (just/Makefile/scripts).
- Otherwise run the standard set for the stack:
  - format
  - lint/static analysis
  - unit tests
  - integration/e2e tests when relevant
- CI is the source of truth. Don’t merge on red.

## Security & privacy
- Treat external inputs as hostile.
- No secrets in code, logs, or screenshots.
- Prefer least privilege and safe defaults.

## Before you finish
- List commands run + results.
- Summarize changes and where (key files).
- Call out deletions of legacy paths.
- Note any follow-ups or risks. No surprises.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jayteealao)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jayteealao)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
