## adaptive-moe-inference

> - Establish a measured baseline before optimization.

# Contributor Instructions

## Evidence and experiment discipline

- Establish a measured baseline before optimization.
- Keep each experiment versioned, reproducible, reversible and attributable to one hypothesis.
- Separate measured evidence, inference and pending validation; never invent performance or quality results.
- Exact paths must preserve token-level oracle equality and pass their resource gates. Approximate paths must remain explicit opt-ins with a documented rollback to the full profile.

## Repository workflow

- Follow `CONTRIBUTING.md` and Conventional Commits.
- Keep commits small and single-purpose, with verification evidence appropriate to the change.
- Do not commit generated or third-party artifacts until provenance and redistribution rights are established.

## Public-release boundary

- Keep credentials, internal topology, personal data, raw private sessions, machine-specific paths and model weights out of Git.
- Store local credentials only in `.env.local`; the repository ignores `.env.local`, `.local/`, `.claude/`, `tmp/` and local worktree directories.
- Preserve upstream copyrights, licenses and the provenance data in `overlays/manifest.json` when changing imported overlay files.

---
> Source: [haiyan-ai-lab/adaptive-moe-inference](https://github.com/haiyan-ai-lab/adaptive-moe-inference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
