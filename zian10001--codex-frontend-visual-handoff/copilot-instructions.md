## codex-frontend-visual-handoff

> This repository contains the public, self-contained Codex Plugin packaging the

# Repository guidance

This repository contains the public, self-contained Codex Plugin packaging the
`frontend-visual-handoff` Skill.

## Boundaries

- Keep the public skill independent from private project registries, user profiles,
  internal host inventories, and organization-specific policies.
- Do not copy files from an installed or private skill tree. Reimplement generic
  behavior and keep fixtures anonymous.
- Keep the repository root as the single Plugin root. Do not duplicate the Skill
  under a multi-plugin catalog tree.
- Treat Kimi Code and every other external coding client as optional providers.
  Codex-only operation must remain usable without them.
- Do not commit credentials, tokens, real business data, private paths, provider
  sessions, screenshots, or deployment evidence.

## Production safety

- Default to read-only inspection. Creating a branch or worktree requires an
  explicit prepare request after inspection.
- Production scripts must not push, pull, fetch, reset, clean, stash, remove,
  unlock, or prune worktrees.
- Never auto-delete a partially created worktree. Return structured evidence and
  preserve the residual for an exact-path audit.
- A reduced-materialization Git worktree is not a security sandbox. Keep that
  limitation visible in user-facing documentation.

## Development and verification

- Support only the compatibility matrix documented in `README.md`; do not infer
  support for another operating system, shell, filesystem, or Git feature.
- Keep test mutations inside a uniquely named temporary directory with a marker
  file and guarded cleanup.
- Use Windows PowerShell-compatible syntax for the Tier 1 implementation.
- Run the Skill and Plugin validators, PowerShell AST parsing, fixture tests,
  generic privacy checks, full-tree text hygiene, any repository-external
  release denylist, and `git diff --check` before proposing a release.
- Do not call or authenticate to an external model provider in automated tests.

---
> Source: [zian10001/codex-frontend-visual-handoff](https://github.com/zian10001/codex-frontend-visual-handoff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
