## git-pre-commit-hook-example

> - The pre-commit hook runs the tests. If it refuses a commit, the code is broken. Fix the code.

# Instructions for coding agents

- The pre-commit hook runs the tests. If it refuses a commit, the code is broken. Fix the code.
- Never use `git commit --no-verify`, `-n`, or `git config --unset core.hooksPath`, and never edit or delete anything under `.githooks/`. Bypassing the hook is not a fix.
- Never commit with tests failing or skipped. A commit is a claim that the tests pass.

---
> Source: [ashleydavis/git-pre-commit-hook-example](https://github.com/ashleydavis/git-pre-commit-hook-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
