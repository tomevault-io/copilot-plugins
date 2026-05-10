## irona

> Ratatui TUI tool for reclaiming disk space from build artifacts. Supports Rust, Node.js, and C# (v1). See `docs/superpowers/specs/2026-04-20-irona-design.md` for full design spec.

# irona — Claude Instructions

## Project

Ratatui TUI tool for reclaiming disk space from build artifacts. Supports Rust, Node.js, and C# (v1). See `docs/superpowers/specs/2026-04-20-irona-design.md` for full design spec.

## Git Workflow

- **No worktrees** — Rust builds are slow; worktrees make it worse
- **One branch per task** — single commit, then open a PR via `gh pr create`
- **Branch stacking** — if follow-on work depends on an unmerged branch, branch from that branch (not main)
- **No mid-task commits** — commit only when the task is complete

## Code Style

- Rust only — no scripts, no extra languages
- No comments unless the WHY is non-obvious
- No placeholder code or TODO stubs in committed work

---
> Source: [kunjee17/irona](https://github.com/kunjee17/irona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
