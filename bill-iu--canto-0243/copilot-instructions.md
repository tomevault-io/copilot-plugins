## dev-branch

> Default git branch for agent work — commit and push to dev, PR to main only when confirmed.


# Dev branch workflow

All code changes from Cursor or Grok Build go to **`dev`**, not **`main`**.

Before editing:

1. `git checkout dev` and `git pull origin dev` (create `dev` from `main` and push if missing).
2. Stay on `dev` for the whole task.

After changes:

1. Commit on `dev`.
2. `git push origin dev`.
3. Do **not** push to `main` or commit on `main`.

Merge to `main` only when the user confirms — open a PR `dev` → `main` (`gh pr create --base main --head dev`). See `docs/agents/git-workflow.md`.

---
> Source: [bill-iu/Canto-0243](https://github.com/bill-iu/Canto-0243) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
