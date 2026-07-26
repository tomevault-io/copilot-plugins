## android-disassembler

> - Maintain and modernize the repository without changing its reverse-engineering focus.

# Android-Disassembler Agent Guide

## Purpose
- Maintain and modernize the repository without changing its reverse-engineering focus.
- Prioritize restoring a reliable baseline before implementing backlog features.
- Treat issue resolution as a combination of code fixes, regression tests, triage, and justified closure.

## Branch And Push Rules
- Start maintenance work from `master` on a local branch prefixed with `codex/`.
- Keep release workflow work separate from baseline maintenance unless explicitly merged.
- Do not push, merge, or tag releases unless explicitly requested.

## Execution Priorities
1. Keep the project buildable and testable.
2. Fix policy and platform compatibility blockers.
3. Land small, high-confidence issue fixes that already have clear repro or existing branch work.
4. Defer broad feature requests into documented backlog buckets.

## Issue Triage Rules
- `baseline-blocked`: cannot address the issue until the build, test, or platform baseline is restored.
- `needs-repro`: insufficient repro data; attempt local repro before changing code.
- `covered-by-open-pr`: there is already a local or remote branch that addresses the issue and should be reviewed first.
- `obsolete-or-policy-invalid`: request is outdated, duplicated, no longer valid on modern Android, or outside the core scope.
- `planned-fast-follow`: valid request, but intentionally deferred until the baseline and policy work are finished.

## Repro And Fix Discipline
- Reproduce bugs before fixing them whenever a reliable repro is possible.
- Add a failing automated test before production code for behavior changes and bug fixes.
- Avoid mixed-purpose commits; baseline recovery, policy changes, and issue-specific fixes should stay separable.

## Android Modernization Rules
- Prefer Storage Access Framework and `ActivityResultContracts` over legacy storage access.
- Avoid introducing new broad storage or package visibility permissions without a documented need.
- Prefer repo-local tests and deterministic fixtures over device-specific assumptions.

## Documentation Rules
- Public-facing docs must use repo-relative paths only.
- Do not include local absolute paths, usernames, SDK install paths, or machine-specific identifiers.
- When recording examples from issues or crashes, remove personal data, package-private paths, and any attached sample references that could identify a reporter.

## Non-Goals For The Baseline Wave
- No Play Store submission yet.
- No push to remote.
- No broad feature expansion until build, tests, and Android policy alignment are in place.

---
> Source: [yhs0602/Android-Disassembler](https://github.com/yhs0602/Android-Disassembler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
