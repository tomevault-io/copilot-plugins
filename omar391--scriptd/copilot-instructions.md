## scriptd

> <!-- markdownlint-disable MD025 -->

<!-- markdownlint-disable MD025 -->
<!-- BEGIN rules:spec:common -->

# Shared Rules

- **Worktree isolation:** never edit, stage, or commit directly on `main`; use a dedicated task worktree first.
- Put all code changes in `<repo>/worktrees/<name>/` and switch before editing.
- Keep the repo-owned `/worktrees/*` path gitignored.
- Follow any runtime skill's worktree naming, branch naming, or reconcile flow.
- One worktree = one coherent task.
- Remove temporary worktrees and branches after landing.
- Use per-worktree tool envs (`bin/`, `.venv/`, `.codex-rotate/bin/`).
- Maintain a real, gitignored `<worktree>/tmp/` agent workspace; before editing, ensure `tmp/tasks.md` and `tmp/plan.md` exist and `git check-ignore tmp/tasks.md tmp/plan.md` passes.
- Treat `tmp/tasks.md` as the canonical execution ledger for the current worktree; do not land while any item remains incomplete.
- If `tmp/plan.md` is missing or empty, create or update it before implementation; use Codex Plan mode when available, otherwise write the plan directly.
- Keep heavy/generated temporary assets outside the repo or inside ignored `tmp/`; never commit `temp`, `tmp`, `_temp`, `_tmp`, `.tmp`, or `.temp` paths.
- Run relevant tests/builds/checks before landing.
- **Token discipline:** no user-facing prose unless needed to complete the requested action. Avoid mid-task updates unless blocked or coordination-critical. Act from context; ask only when needed. End with compact `what changed / how / checks` when applicable. Omit logs, diffs, and repeated context unless requested.

<!-- END rules:spec:common -->
<!-- BEGIN rules:spec:coding -->

# Coding Baseline

- Default to the `mre` skill: build for the current proven need; choose the highest safe rung: no change, deletion, reuse, platform/stdlib, installed dependency, new code, then new dependency.
- Keep edits scoped and follow repo idioms.
- Prefer TDD/BDD: write or update tests before (or alongside) the implementation for behavior changes. Apply SOLID only when it reduces churn.
- **Integration tests must use isolated live environments** (sandboxed databases, test accounts, ephemeral services). Never run integration tests against a production runtime or data store.
- Optimize for agentic locality: prefer cohesive production files ~300-500 lines; treat 500+ as a smell and 1,000+ as a split candidate in multi-file modules unless generated, declarative, or inherently cohesive.
- Split by semantic boundary, for example types, I/O, validation, domain logic, UI state/view, CLI parsing/execution, test helpers, or test scenarios.
- Avoid splits where the pieces must always be read or changed together.

<!-- END rules:spec:coding -->
<!-- BEGIN rules:local -->
<!-- END rules:local -->

Load on-demand specs: [`code-review`](~/.agents/skills/agent-md/assets/specs/code-review.md), [`ts`](~/.agents/skills/agent-md/assets/specs/ts.md)

---
> Source: [omar391/scriptd](https://github.com/omar391/scriptd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
