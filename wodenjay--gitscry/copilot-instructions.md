## gitscry

> - If you need to create a worktree, save it to ./.worktrees. Ask me before using worktrees.

# GitScry

## Development Rules

- If you need to create a worktree, save it to ./.worktrees. Ask me before using worktrees.
- After each logically self-contained change, stage and create a new commit. Follow /caveman-commit skill for commit messages.
- If /implement skill is used, after finishing the whole task: reviewer then find bugs -> add test (if possible; **never write complex and meaningless test**) -> RED -> fix bugs -> GREEN -> done. **Do NOT re-review after fixing bugs. One task, one batch review**.
- Keep code clean and intentional with /codebase-design skill; prefer clarity over compatibility. Give each module one reason to change, with a small public interface over a private implementation.

## Cargo Rules

- Limit cargo thread: `--jobs 1`.
- Before completing a Rust task, run `cargo fmt --all -- --check` and `cargo clippy --all-targets --all-features --jobs 1 -- -D warnings`. No warnings should be emitted.

## Agent skills

- Issue tracker: Issues live as GitHub issues in `WodenJay/GitScry`. See `docs/agents/issue-tracker.md` and use /gh-cli skill.
- Triage labels: The five canonical triage roles map to identically named labels (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). See `docs/agents/triage-labels.md`.
- Domain docs: Single-context: `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

---
> Source: [WodenJay/GitScry](https://github.com/WodenJay/GitScry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
