## commit-style

> When asked to generate a commit log, please adhere to the following guidelines:

# Commit Message Generation Rule

When asked to generate a commit log, please adhere to the following guidelines:

1.  **Language:** All commit messages must be written in English.
2.  **Style:** Follow the "Conventional Commits" specification. Use a type prefix like `feat:`, `fix:`, `perf:`, `chore:`, etc., followed by a concise, lowercase summary.
3.  **Reference:** To ensure consistency, refer to the user's recent commit history to match their established style. You can use a command like `git log --author="Lysander" --pretty=oneline -n 10` for guidance.
4.  **Content Analysis Workflow:** Base the commit message on the actual code changes. The standard workflow should be:
    *   First, use a `git status` equivalent to see which files were changed and understand the scope.
    *   Second, use a `git diff` equivalent to analyze the specific code modifications to understand the purpose and details of the changes.

---
> Source: [Lysander66/briefy](https://github.com/Lysander66/briefy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
