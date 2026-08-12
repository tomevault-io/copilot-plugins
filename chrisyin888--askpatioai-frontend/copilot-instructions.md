## auto-push

> Auto commit and push after completing implementation work in this repo


# Auto push after changes

When you finish a user-requested task that modified tracked project files:

1. **Commit and push automatically** — do not wait for the user to say "push".
2. Follow the git safety protocol (no force push, no secrets, no amending pushed commits, HEREDOC commit messages).
3. **Exclude by default**: untracked `docs/`, `.env`, credentials, and files the user said not to commit.
4. **Skip auto push** only when:
   - The task was question-only or review-only (no code changes)
   - The user explicitly says not to push / not to commit
   - Commit would be empty
5. Push to the **current branch** (usually `main`) with `-u` if no upstream is set.
6. After push, briefly tell the user the commit hash and what was included.

---
> Source: [chrisyin888/askpatioai-frontend](https://github.com/chrisyin888/askpatioai-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
