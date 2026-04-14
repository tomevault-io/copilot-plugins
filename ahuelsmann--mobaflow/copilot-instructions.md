## mobaflow

> MOBAflow project context, real project names, build constraints,

description: >
  MOBAflow project context, real project names, build constraints,
  language, and safe working style.
trigger: always_on
---

<!-- markdownlint-disable MD003 MD041 -->

# MOBAflow Project Context

- This repository is a `.NET 10` multi-project solution for model railroad automation.
- Use the current project names from the repository, not outdated doc aliases:
  - `MOBAflow` = WinUI desktop app
  - `MOBApi` = REST API / web backend
  - `MOBAsmart` = MAUI Android app
- Core cross-platform layers are `Domain`, `Common`, `Backend`,
  `SharedUI`, `Sound`, `TrackLibrary.Base`, `TrackLibrary.PikoA`, and
  `TrackPlan.Renderer`.
- Respect the dependency direction:
  - `Domain` -> `Backend/Common` -> `SharedUI` -> platform apps
- Use the SDK pinned in `global.json`:
  - `.NET SDK 10.0.103`
- Do not assume solution-level restore/build is the safest path for all environments.
  - Prefer targeted `dotnet restore <project>.csproj`
  - Prefer targeted `dotnet build <project>.csproj`
  - Standard test entry point is `dotnet test Test/Test.csproj`
- The user communicates in German.
- Code comments and XML documentation should be written in English.
- Keep changes small, reviewable, and easy to reason about.
- Do not perform Git write operations such as commit, push, merge,
  rebase, or cherry-pick.
- Suggest relevant tests when behavior changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahuelsmann) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
