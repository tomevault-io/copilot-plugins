## auto-push

> Commit and push to GitHub Pages after each completed code update


# Auto-push after updates

When the user requests code changes and you finish implementing them:

1. Verify the build passes (`npm run build`) when changes are non-trivial.
2. Commit with a concise message following existing repo style (why, not just what).
3. Push to `origin main` without waiting for a separate "push" request.
4. Briefly tell the user the commit hash and that GitHub Pages deploy is running.

## Git safety

- Follow the standard commit protocol (status, diff, log before committing).
- Do not commit secrets (`.env`, credentials).
- Do not force-push to `main`.
- Do not amend unless the standard amend rules apply.

## Exceptions

- Do not push if the user explicitly says not to, or only asked a question with no code changes.
- Do not push if the build fails; fix first, then commit and push.
- Do not create empty commits.

Deploy target: https://gking432.github.io/ToDoToday/

---
> Source: [gking432/ToDoToday](https://github.com/gking432/ToDoToday) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
