## pr-conflict-check

> Always check for and resolve merge conflicts before declaring a PR shippable


# PR Conflict Check

Before opening a PR or declaring a branch shippable, always:

1. Fetch the latest main: `git fetch origin main`
2. Check mergeability: `gh pr view <number> --json mergeable,mergeStateStatus`
   - Or before the PR exists: `git merge-tree $(git merge-base HEAD origin/main) HEAD origin/main`
3. If `mergeable: "CONFLICTING"`, rebase onto main: `git rebase origin/main`
4. Resolve every conflict file manually - keep both sides when content is additive (e.g. two execution ledgers in the same file), take the correct version when one side is stale
5. `git add <resolved>` then `GIT_EDITOR=true git rebase --continue`
6. `git push --force-with-lease` to update the remote branch
7. Re-check `gh pr view --json mergeable` until it returns `"MERGEABLE"`

## Never skip this step

A PR opened with `mergeStateStatus: "DIRTY"` cannot be merged and blocks the team. Always resolve conflicts before handing off for review.

---
> Source: [legioncodeinc/that-git-life](https://github.com/legioncodeinc/that-git-life) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
