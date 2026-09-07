## git-after-phases

> Commit and push after each shipped phase or feature


# Git after phases

When a phase or user-requested feature is fully implemented (tests passing, FEATURES.md updated), create a git commit and push to `origin` without waiting to be asked again.

- Do not commit `.env`, databases, or `uploads/`
- Do not force-push `main` unless the user asked to rewrite history
- Follow the existing commit-message style: 1–2 sentences on why
- Author and committer must be the user's git identity. Do not put tool names in commit messages. If a `Co-authored-by` trailer is added automatically, rewrite the commit with `git commit-tree` before pushing.

---
> Source: [Gourav-praneeth/GradeLens](https://github.com/Gourav-praneeth/GradeLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
