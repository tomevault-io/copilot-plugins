## homelab-infrastructure-portfolio

> Portfolio repository containing sanitized artifacts synced from development projects (RAG pipeline, HN digest, etc.).

# CLAUDE.md - homelab-portfolio

## Project Overview

Portfolio repository containing sanitized artifacts synced from development projects (RAG pipeline, HN digest, etc.).

## Testing

- After any code changes, always run the full test suite with `pytest -v --tb=short` and confirm zero regressions before committing.
- Report pass/fail counts explicitly.
- Never skip or delete tests to make the suite pass.

## Git Conventions

- Always use `git add`, `git commit`, and `git push` as the final step of a session.
- Use `cp -f` when copying/overwriting files during sync operations.
- Commit messages should be descriptive and reference the phase/session when applicable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michelmazza) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
