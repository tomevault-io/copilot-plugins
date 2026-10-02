## tollgate

> Build spec: `plan.md`. Requirements: `prd.md`. Design: `architecture.md`.

# Tollgate

Build spec: `plan.md`. Requirements: `prd.md`. Design: `architecture.md`.

## Rules

### Git: never touch it

NEVER run `git add`, `git commit`, `git push`, `git reset`, `git revert`, `git checkout`,
`git switch`, `git stash`, `git merge`, `git rebase`, `git tag`, `git branch`, or `git clean`.
Not once, not "to be helpful", not even when a task in `plan.md` shows a commit message.

The commit messages in `plan.md` are for the human to use. You do not commit. The human commits
every change themselves, after reviewing it.

Read-only git is fine: `git status`, `git diff`, `git log`, `git show`.

When a task is finished, stop and say "Task N.M complete, ready for you to review and commit",
then show `git diff --stat` and the suggested commit message as text. Then wait.

### Everything else

- Execute `plan.md` one task at a time. Never start the next task without being asked.
- Never skip a task's Verification block. Run the commands. Paste the real output.
  Do not claim a verification passed without running it.
- Never move to the next phase until the Phase Gate passes.
- Implement exactly the task's scope. Anything you notice but that is not in scope goes in a
  `NOTES.md` "Deferred" list, not into the diff.
- Do not build anything in the "Post-MVP / Out of Scope" section of `plan.md`.
- If the task's Implementation section is wrong or impossible given the actual code, stop and
  say so before writing anything.

## Stack

Rust workspace (gateway + settler + core), Foundry contracts, Python client, Next.js dashboard.
Base Sepolia (84532), native USDC only. `alloy` for all signature work.

## Checks

`cargo clippy --workspace -- -D warnings` and `cargo test --workspace` must pass before any commit.
Later phases add `forge test`, `pytest`, and `npm run build` via `./scripts/check.sh`.

---
> Source: [sanjayrohith/Tollgate](https://github.com/sanjayrohith/Tollgate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
