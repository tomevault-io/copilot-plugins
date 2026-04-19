## sandboxtest

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Session Start — Read Memory Files First

At the start of every session, read all files in `.claude/memory/` before doing anything else:

- `.claude/memory/project.md` — phase status, goals, design decisions, run commands
- `.claude/memory/architecture.md` — full file tree, service responsibilities, API endpoints
- `.claude/memory/design-system.md` — CSS variables, class names, fonts, Bootstrap usage
- `.claude/memory/blazor-patterns.md` — page template, polling pattern, Razor quoting rules, pitfalls
- `.claude/memory/phase2.md` — Azure SDK integration plan (not started)
- `.claude/memory/known-issues.md` — bugs, gotchas, fixed issues

These files are the authoritative source of truth for project context. Always check them before reading source files.

## Commands

```bash
dotnet build                              # compile check (run after any code change)
dotnet run --project SandboxTest          # dev server — http://localhost:5169
dotnet watch --project SandboxTest        # hot reload
dotnet test                               # run tests (if test project added)
```

## Architecture Summary

Three-project .NET 10 Blazor Web App (Auto interactivity):

| Project | Role |
|---------|------|
| `SandboxTest/` | Server — Blazor pages, layout, minimal API endpoints, server-side services |
| `SandboxTest.Client/` | WASM — HttpClient-based API clients only (no pages) |
| `SandboxTest.Shared/` | Shared models + interfaces (no external dependencies) |

All interactive pages use `@rendermode InteractiveAuto` and live in the **server** project. The client project only provides API clients registered via `IServiceCollection`. See `.claude/memory/architecture.md` for the full file tree.

## Key Rules

- Never nest `@{}` blocks inside `@if`/`@else` in Razor — declare locals outside the block
- Never use `"..."` inside `@()` within double-quoted HTML attributes — use local variables
- `DeploymentStatus` enum: `Pending`, `Running`, `Succeeded`, `Failed`, `Cancelled`
- `ParameterType` enum: `String`, `Int`, `Bool`, `Select`
- All API responses return `ApiResponse<T>` — check `.Success` before using `.Data`
- `SandboxTest/data/` is gitignored (runtime state, not source)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/acutraaq) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
