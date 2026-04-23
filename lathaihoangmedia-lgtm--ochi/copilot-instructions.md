## ochi

> This file aligns Claude Code behavior with the shared repository standard.

# Claude Code Instructions for Ochi

This file aligns Claude Code behavior with the shared repository standard.

## Source of Truth

- Follow repository-wide standards in AGENTS.md.
- Follow Copilot/AI Builder workflow constraints in .github/copilot-instructions.md.

## Required Execution Style

- Read relevant code first, then implement.
- Keep changes minimal and scoped.
- Validate before handoff.
- Report blockers with exact command output.
- Keep changes modular for Rust + Go + Rust2Go integration paths.
- Prefer lightweight files/functions (target <= 300 lines per file, <= 60 lines per function).

## Validation Baseline

For Rust changes, attempt:

```bash
cargo check --workspace
cargo test --workspace
```

If environment restrictions block full checks, run narrower checks and clearly explain why.

## Handoff Format

Always include:

1. What changed
2. Where changed
3. Validation run and result
4. Remaining risk or next step

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lathaihoangmedia-lgtm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
