## pz

> Build `pz`: a Zig CLI harness with TUI. Do not optimize for compatibility with pi.

# Agent Rules

## Mission

Build `pz`: a Zig CLI harness with TUI. Do not optimize for compatibility with pi.

## Core Constraints

- Correctness over speed.
- Use hard cutovers only: no legacy support, compatibility shims, or fallback paths.
- No silent fallbacks.
- Root-cause fixes only.
- Keep names short and clear.
- Keep hot paths allocation-aware.

## Read Window Rule

- Use `rg -n` to find the anchor first.
- Default file reads to a 20-line window around the anchor.
- If that is insufficient, double the window: 40, then 80, then 160.
- Only start wider when it is obvious up front that 20 lines cannot contain the relevant construct.

## Trace Rule

- Emit terse commentary traces during active work so the user can see progress.
- Include a trace before substantial reads, edits, test runs, merges, and agent launches.
- Keep traces factual and short.

## Source Control

Use `jj`, not `git`.

- New change: `jj new`
- Describe change: `jj describe -m "<imperative message>"`
- Sync: `jj git fetch` / `jj git push`

## Parallel Work (Required)

For multi-agent work, use separate `jj` workspaces.

Default shape:

1. Keep `1` local lane for integration, rebases, merges, flakes, and shared-surface debugging.
2. Launch only `2-3` worker lanes by default.
3. Expand past `3` workers only if the next dots are truly disjoint in files and dependencies.

1. Create workspaces under `.jj-ws/` (see jj skill for commands).
2. Assign file ownership per workspace.
3. Do not edit files owned by another workspace.
4. Reconcile by rebasing/squashing after each track stabilizes.

## File Ownership Rule

If a file is touched by another active agent, stop and reassign before editing.
Do not launch overlapping dots in parallel just because they look “small”.
If dots overlap in runtime/provider/session/core surfaces, serialize them unless a prior dot closes the dependency.

## Commit Rule

Only include files changed by the current agent/task. No broad staging.

## Agent Reporting Rule

Agents must report what they CHANGED (files, lines, insertions/deletions), not claim things were "already done." If no changes needed, explain why with file:line evidence. Never declare victory without a diff.

## Per-Dot Verification Gate (BLOCKING)

NEVER merge a workspace without verification. After each agent completes:

1. Launch a **verification agent** (Opus, fresh context) that receives:
   - The plan item's acceptance criteria (from PLAN.md)
   - The workspace diff (`jj diff -r <ws-commit>`)
   - Full content of every file touched by the diff (not just hunks)
   - Test results
2. Verifier must:
   - Check EACH acceptance criterion: PASS/FAIL with file:line evidence
   - Name the specific test that exercises each criterion
   - List at least ONE thing the diff does NOT address or could break
   - Check for shortcuts: new `catch {}`, `catch return null`, `std.log.warn` replacing propagation, `unreachable`
3. Merge only on full pass. On any fail: fix in workspace, re-verify.
4. Batch verification (3-5 dots) is acceptable to amortize cost.

## Testing Rule

Run `timeout 60 zig build test 2>&1 | tail -5` ONCE. Never loop, retry, or run multiple grep variations.
A useful test proves behavior the compiler cannot — error paths, edge cases, integration contracts, security boundaries. Skip tests that verify comptime-guaranteed behavior, constants, or type correctness.
Run relevant tests before and after each fix or feature.
Every bug fix must add or strengthen a test.
Use `ohsnap` snapshots for struct/multi-field outputs and serialized payload checks.
Use `std.testing.expectEqual` only for scalar primitives.

### UX Testing (two layers, BOTH required per UX row)
1. **Mock-terminal (fast, <1s)** — construct Ui/vscreen in memory, drive with method calls. Catches rendering/state regressions. Runs every build.
2. **PTY keyboard (slow, 2-5s)** — spawn real `pz` binary via `src/test/pty_harness.zig`, inject keystrokes, verify screen via vscreen/ANSI. Proves the real binary works from the keyboard.
A UX flow is NOT fully tested until BOTH layers exist. The walkthrough matrix requires PTY tests.
Use `joelreymont/zcheck` for property tests and add fuzz/property coverage where the surface warrants it.

## Zig Rules

See `docs/zig.md` (mirrored from `~/.agents/docs/zig.md`).

## Formal Verification

- TLA+ spec for thread dispatch: `docs/tla/ThreadDispatch.tla`
- Run with `/tla` skill after changing thread dispatch, file ownership, or join/cancel logic
- Lean 4 proofs planned for policy enforcement and signing (see dot)

## Plan Rule

Track execution against `PLAN.md`.
When a plan item is complete, update status in commit message and notes.

## Lessons Rule

Read `LESSONS.md` at the start of work.
Update `LESSONS.md` at the end of the session with new do-more and do-not-do lessons.

## Release Rule

For release work, import and follow `.claude/skills/release/SKILL.md` in addition to this file.
Release prep must include a `CHANGELOG.md` entry for the new version before tagging.

---
> Source: [joelreymont/pz](https://github.com/joelreymont/pz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
