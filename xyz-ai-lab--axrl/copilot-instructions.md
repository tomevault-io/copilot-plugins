## axrl

> Do not stage, unstage, commit, amend, reset, or otherwise modify the Git index

# Agent Instructions

## Git Safety

Do not stage, unstage, commit, amend, reset, or otherwise modify the Git index
or Git history unless the user explicitly asks for that action in the current
conversation.

Do not run commands such as:

- `git add`
- `git rm --cached`
- `git commit`
- `git reset`
- `git restore --staged`
- `git checkout --`

Editing files is allowed when requested, but staging and committing are the
user's responsibility.

If Git state matters, inspect it with read-only commands such as `git status`,
`git diff`, and `git diff --cached`.

---
> Source: [XYZ-AI-Lab/axrl](https://github.com/XYZ-AI-Lab/axrl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
