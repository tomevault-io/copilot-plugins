## agentloop

> This file applies to any agent working in this repository (Cursor, Claude Code, Codex, etc.).

# Agent Loop — instructions for coding agents

This file applies to any agent working in this repository (Cursor, Claude Code, Codex, etc.).

## Before every push

**You must bump the root `VERSION` file before pushing to GitHub.**

The kit uses `VERSION` as the release identifier: installs copy it to `.agent-loop/kit-version`, and `agent-update-check` compares it with the remote file on GitHub.

### How to bump

1. Read the current value in [`VERSION`](VERSION) (single line, semver, e.g. `0.1.0`).
2. Increment it **before** `git push`:
   - **Patch** (`0.1.0` → `0.1.1`): default for bug fixes, GUI tweaks, docs, and most kit changes.
   - **Minor** (`0.1.0` → `0.2.0`): new features or behavior changes that are backward compatible.
   - **Major** (`0.1.0` → `1.0.0`): breaking changes to install layout, CLI contract, or queue/task format.
3. Write only the new version string in `VERSION` (no `v` prefix, no extra lines).
4. Include `VERSION` in the same commit you push, or in a dedicated release commit immediately before push.

### Do not push without a version bump

If the branch has user-facing or kit changes that will land on `master`, **do not push** until `VERSION` is updated. Skipping this breaks update notifications for downstream repos.

### Quick checklist

- [ ] Changes committed
- [ ] `VERSION` incremented
- [ ] `git push` (only when the user asked to push)

## Related docs

- [`README.md`](README.md) — install, workflow, backends
- [`adapters/cursor/skills/update/SKILL.md`](adapters/cursor/skills/update/SKILL.md) — update flow and Agent Console restart

---
> Source: [ciroaurelio22/AgentLoop](https://github.com/ciroaurelio22/AgentLoop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
