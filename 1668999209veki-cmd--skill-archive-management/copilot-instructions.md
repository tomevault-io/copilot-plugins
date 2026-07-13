## skill-archive-management

> This repository's default user-facing output is the Skill archive dashboard: `skill-archive.html`.

# Codex Instructions

This repository's default user-facing output is the Skill archive dashboard: `skill-archive.html`.

When a user installs, clones, opens, or asks how to use this repository, guide them to refresh and open the dashboard first:

- Windows: run or double-click `launchers/windows/start-dashboard.cmd`.
- macOS: run or double-click `launchers/macos/start-dashboard.command`.
- CLI fallback: run `powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\setup-dashboard.ps1`.

Do not run `scripts/build-skill-source-index.ps1` or `scripts/check-skill-updates.ps1` as the default install flow. Those scripts are optional maintenance tools and can generate separate index/report files. Use them only when the user explicitly asks for source indexing or update health checks.

To show the user's own local data, the dashboard must be refreshed on the user's machine. Opening the bundled `skill-archive.html` without refreshing shows the repository snapshot, not necessarily the user's local Codex/Agent Skills.

---
> Source: [1668999209veki-cmd/Skill-Archive-Management](https://github.com/1668999209veki-cmd/Skill-Archive-Management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
