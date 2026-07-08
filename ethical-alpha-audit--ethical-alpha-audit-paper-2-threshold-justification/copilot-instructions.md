## base-git-discipline

> Git commit, branch, and push discipline


# Git discipline

- **One logical change per commit.** Do not bundle "fix Sobol numbers + update manuscript + reformat README" into one commit.
- **Commit messages start with a verb.** "Update Sobol primary N to 2048" not "Sobol updates".
- **Commit messages name the canonical reference where applicable.** "Implement Stage 3.5 §6 DGP per Att-06" not "add DGP".
- **No force-pushes to main.** Use feature branches if the change is non-trivial; merge via PR even on solo work, for the audit trail.
- **MANIFEST.sha256 (where present in a paper repo) is regenerated automatically before any commit that changes outputs/ or reproducibility/.** This is the per-paper analogue of the v8.5 RECORD action; it records state, it does not gate progression.
- **Push after each logical commit.** Don't accumulate 8 commits and push in a batch — the GitHub state should track local state continuously.

---
> Source: [ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification](https://github.com/ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
