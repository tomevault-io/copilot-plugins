## token-map

> Maintain `TokenMap` as a local desktop app for source-tree analysis with synchronized tree and treemap inspection.

# AGENTS.md

## Goal
Maintain `TokenMap` as a local desktop app for source-tree analysis with synchronized tree and treemap inspection.

## Working Baseline
- Windows is the primary MVP target; macOS is secondary validation; do not spend the current task on Linux polish.
- Current technical boundaries live in `docs/architecture.md`.
- Day-to-day execution flow and active-plan handling live in `docs/workflow.md`.
- See `docs/localization.md` for localization.

## Non-Negotiables
- `Clever.TokenMap.Core` stays free of Avalonia.
- `Clever.TokenMap.App` does not read the file system directly; it goes through services and contracts.
- `MainWindowViewModel` stays a shell coordinator; long-lived analysis, settings, and treemap scope state live in app-layer services/state objects.
- Token counting stays local behind `ITokenCounter`.
- Line metrics stay local in the infrastructure analysis pipeline for included text files.
- The scanner defines the included tree and the set of analyzed nodes.
- The treemap stays one custom-rendered control, not a control-per-rectangle surface.
- Do not add WebView, browser embedding, JS charting, cloud services, remote tokenizers, or heavy dependencies without explicit need.

## Change Rules
- Do not make unrelated refactors.
- Do not change architecture decisions without updating the current-state docs.
- Keep one canonical representation per concept; do not retain legacy aliases, compatibility shims, or historical fallback paths unless the user explicitly requires backward compatibility.
- In PowerShell, do not use `&&`; use compatible command separation.
- Follow `docs/commit-policy.md` for commit strategy and git safety.

## Documentation Rules
- Keep docs limited to current state and plans.
- Keep one source of truth: rewrite the canonical instruction in place, prefer fewer precise rules, and do not add duplicate or follow-up guidance beside it.
- Update `docs/AGENTS.md` if the documentation map or policy changes.

## Verification
- Standard repo verification flow, run sequentially and never overlap `dotnet build` with `dotnet test`:
  - `dotnet restore`
  - `dotnet build Clever.TokenMap.sln`
  - `dotnet test Clever.TokenMap.sln --no-build`
- If `dotnet build` or `dotnet test` hits a file lock from a running app process, first stop the currently running `Clever.TokenMap.App` instance and then rerun the blocked verification step.
- If the change adds headless or UI tests, run them too.

## If Blocked
1. Choose the smallest workaround that does not break the documented architecture.
2. If the blocker creates a lasting repo limitation, reflect it in the relevant current-state doc.
3. Do not expand scope unless necessary.

---
> Source: [etechlead/token-map](https://github.com/etechlead/token-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
