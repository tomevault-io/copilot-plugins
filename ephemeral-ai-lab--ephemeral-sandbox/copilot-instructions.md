## ephemeral-sandbox

> Agent guidance for this repository. Follow `CLAUDE.md` for the full project

# AGENTS.md

Agent guidance for this repository. Follow `CLAUDE.md` for the full project
rules.

## Required rules

- Always work, commit, and push directly on `main`. Do not create side branches
  (including `agent/*`) or additional git worktrees.
- Rebuild the Docker sandbox gateway binary with
  `bin/start-sandbox-docker-gateway --rebuild-binary`.
- Use `sandbox-manager-cli`, `sandbox-runtime-cli`, and
  `sandbox-observability-cli` for manual sandbox operations.

---
> Source: [Ephemeral-AI-Lab/ephemeral-sandbox](https://github.com/Ephemeral-AI-Lab/ephemeral-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
