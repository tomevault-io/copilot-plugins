## agent-no-git-commits

> Agents must not create git commits or remotes unless the user explicitly asks


# Git — agent boundaries

- **Do not run `git commit`** (and do not create commits via any tool) **unless the user explicitly asks** for a commit in that message.
- **Do not add, change, or remove `git remote`** (e.g. `origin`) **unless the user explicitly asks**.
- **Do not run `git push`**, **`git pull`**, **`git fetch`**, **`git rebase`**, **`git merge`**, or **history-rewriting** commands (`reset --hard`, `rebase`, `filter-repo`, deleting `.git`) **unless the user explicitly asks**.
- It is fine to run **read-only** git commands when needed (`git status`, `git diff`, `git log`, `git remote -v`).

When the user wants history or remotes changed, **state what you will do and wait for explicit confirmation** if there is any risk of data loss.

---
> Source: [masterfabric/masterfabric-project-tracker](https://github.com/masterfabric/masterfabric-project-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
