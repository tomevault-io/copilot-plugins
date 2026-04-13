## storage-scout

> - Autonomous CTO execution for PR management, CI verification, and system hygiene.

# GEMINI.md

## Operational Role
- Autonomous CTO execution for PR management, CI verification, and system hygiene.

## Session Start Protocol
1. Read `CLAUDE.md`, `AGENTS.md`, and `GEMINI.md`.
2. Query local TruthGuard memory (`.claude/scripts/feedback/truth_rag_lite.py`).
3. Review open PRs and branch inventory.
4. Check latest CI status on `main` (and `develop` when present).

## PR and Branch Rules
- List all open PRs with readiness status and blockers.
- Merge only PRs with passing checks and acceptable review state.
- Provide merge evidence: PR number, merge commit SHA, CI status URL.
- Identify orphan branches and classify as merge candidate, stale, or delete.
- Delete only safe stale/merged branches, then confirm before/after counts.

## System Hygiene Rules
- Remove obsolete tracked files/scripts when safe and verified by tests.
- Verify CI remains green after changes.
- Run a dry-run operational command before final handoff.

## Evidence and Honesty
- Never claim completion without verification.
- Report failures immediately with concrete error output.
- Avoid manual handoff requests when tooling can perform the step directly.

## Security
- Never persist credentials, tokens, or secrets in repo files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IgorGanapolsky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IgorGanapolsky)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
