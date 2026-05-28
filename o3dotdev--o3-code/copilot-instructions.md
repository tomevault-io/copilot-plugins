## o3-code

> Repo-local skills live under `.agents/skills/`. Use these before global copies when the task matches the skill.

# Repository Guidelines

## Agent skills

Repo-local skills live under `.agents/skills/`. Use these before global copies when the task matches the skill.

- `codex-app-source-refresh`: refresh O3 Code from a newer installed Codex App release while preserving the Source Refresh, Normalization, Patch SOP, and Patch Evidence workflow.
- `mirrored-web-client-refresh`: rebuild the Mirrored Web Client Asset Tree and reapply browser-only Web Patch SOPs after Desktop Reconstruction patches are applied.

### Issue tracker

Issues and PRDs for this repo are tracked in GitHub Issues for `o3dotdev/o3-code`. See `docs/agents/issue-tracker.md`.

### Triage labels

Use the default triage label vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, and `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

This is a single-context repo with one root `CONTEXT.md` and ADRs under `docs/adr/` when durable decisions need to be recorded. See `docs/agents/domain.md`.

## Project context

This repo is for reconstructing the installed macOS Codex App into a repo-local Electron project that can be installed, formatted, patched, and run through `pnpm`.

Treat the installed macOS app as read-only upstream material. Preserve copied source and assets unless the user explicitly asks for cleanup or replacement.

---
> Source: [o3dotdev/o3-code](https://github.com/o3dotdev/o3-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
