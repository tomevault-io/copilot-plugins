## sema

> If a `../CLAUDE.md` and `../repos.tsv` exist beside this repo, you are inside the

@AGENTS.md

## Sema workspace (if present)

If a `../CLAUDE.md` and `../repos.tsv` exist beside this repo, you are inside the
**sema-lisp workspace** meta-repo, and its `../CLAUDE.md` is MANDATORY here:

- Create/remove git worktrees ONLY via `jake wt-new` / `jake wt-rm` run from the
  workspace root — never `git worktree add` by hand, and never outside
  `../.worktrees/`.
- Rust builds run with incremental compilation on and NO rustc wrapper
  (`../.cargo/config.toml`; policy rationale in `docs/build-time-report.md`).
  Don't re-add an sccache wrapper — it hard-fails incremental builds. Reclaim
  disk with `jake sweep` (worktree hygiene matters more with incremental on).

Read `../CLAUDE.md` before creating worktrees or running large builds.

---
> Source: [sema-lisp/sema](https://github.com/sema-lisp/sema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
