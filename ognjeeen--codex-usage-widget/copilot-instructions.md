## codex-usage-widget

> Codex Usage Widget is a local-only Windows desktop utility that displays Codex usage in a movable WPF widget and beside the Windows notification area. It targets .NET 10 and obtains rate-limit data through the locally installed Codex CLI and its official app-server protocol.

# AGENTS.md

## Project

Codex Usage Widget is a local-only Windows desktop utility that displays Codex usage in a movable WPF widget and beside the Windows notification area. It targets .NET 10 and obtains rate-limit data through the locally installed Codex CLI and its official app-server protocol.

Before changing code, read `README.md`, `docs/ARCHITECTURE.md`, and the files directly involved in the requested behavior. For release work, also read `docs/RELEASING.md`.

## Scope and decision discipline

- Implement the smallest complete change that satisfies the explicit request.
- Preserve behavior outside the requested scope.
- Do not add speculative features, configuration, compatibility layers, dependencies, or refactors for possible future needs.
- Prefer a direct implementation over a generalized system. Introduce an abstraction only when it solves a concrete current problem, protects an existing architectural boundary, or creates a necessary test seam.
- Do not turn an isolated change into a repository-wide cleanup. Mention worthwhile out-of-scope improvements separately instead of implementing them.
- Refactor only when the existing structure directly prevents a safe change. Keep the refactor proportional to the request.
- Keep one cohesive responsibility per type and file, but do not split trivial logic into extra files merely to reduce file size.
- Follow established project patterns unless the task explicitly requires changing them.
- Resolve discoverable, reversible, in-scope implementation details by inspecting the repository and following its existing patterns. Do not stop for minor choices that do not materially affect the result.
- Ask before making a choice that materially changes UX, architecture, public behavior, data handling, packaging, supported platforms, or dependencies.

## Authorization and repository safety

- Treat existing uncommitted changes as user-owned. Do not overwrite, revert, reformat, stage, or include them unless they are explicitly part of the task.
- Do not commit, push, create or update a pull request, tag, publish a release, rewrite history, or otherwise change remote state unless the user explicitly requests that action.
- When the user asks for a local implementation or preview, stop after local validation and wait for approval before any Git or GitHub publishing action.
- Never use destructive Git or filesystem commands to resolve an unrelated workspace problem.
- Keep personal machine paths, usernames, credentials, tokens, and environment-specific values out of tracked files. Use repository-relative paths in documentation and configuration.

## Architecture boundaries

- `Domain/` contains models and domain rules. It must not depend on WPF, Windows Forms, process management, JSON transport details, or Win32 APIs.
- `Application/` coordinates use cases and depends on domain types and ports such as `IUsageProvider`.
- `Infrastructure/` owns Codex protocol integration, process lifetime, settings persistence, logging, and Windows/Win32 integration.
- `Views/` owns WPF rendering and interaction. Do not parse Codex protocol payloads or perform persistence and process-management work in view code-behind.
- Keep Win32 interop in `Infrastructure/Windows/` and UI presentation in `Views/`.
- Preserve the explicit composition root in `App.xaml.cs`. Do not add a dependency-injection package without a concrete requirement.
- Prefer the existing .NET, WPF, and Win32 stack. Add a NuGet dependency only when it solves a concrete current requirement, and verify that its license is compatible with this public repository.
- Keep the Codex CLI as the owner of authentication. Do not read, copy, expose, or log authentication secrets.

## Windows UI changes

- Reproduce interaction bugs before changing focus, activation, hit-testing, z-order, window ownership, or fullscreen behavior.
- Preserve normal taskbar and notification-area behavior; a fix for one Windows shell state must not make the overlay steal focus or block unrelated input.
- When relevant to the change, verify behavior with DPI scaling, multiple monitors, different taskbar positions, and taskbar auto-hide rather than assuming the primary-monitor default.
- Validate visual and interaction changes by running the app on Windows. Do not claim that GUI behavior was verified if only automated tests were run.
- For user-visible UI changes, let the user verify the local build before committing or publishing when the workflow allows it.

## Validation

Run checks proportional to the change. Start with the narrowest relevant test, then use the repository-level checks for code changes:

```powershell
dotnet restore .\CodexUsageWidget.slnx
dotnet build .\CodexUsageWidget.slnx -c Release --no-restore
dotnet test .\CodexUsageWidget.slnx -c Release --no-build
```

- Add or update regression tests for changed domain, application, parsing, settings, or testable Windows-integration behavior.
- For a bug fix, verify both the original failure scenario and the corrected behavior.
- If Windows shell behavior cannot be covered reliably by an automated test, state that limitation and document the manual check performed.
- Documentation-only changes do not require a full build unless they alter commands, packaging, or release instructions.
- Use `scripts/publish.ps1` only when validating distribution or release-sensitive changes; it creates artifacts and is not required for ordinary edits.

## Privacy and product boundaries

- Keep the application local-only unless a task explicitly changes that product decision.
- Use the official local Codex app-server flow. Do not add browser scraping, credential extraction, telemetry, analytics, or a remote backend as an incidental implementation detail.
- Log only what is needed for diagnostics and never log secrets or full sensitive payloads.

## Release discipline

- The project version in `src/CodexUsageWidget/CodexUsageWidget.csproj` is the release source of truth.
- Follow `docs/RELEASING.md`; do not invent a second release path.
- Create or publish a release only after the requested changes are approved and the required CI checks pass.
- Never reuse or move an existing release tag.

---
> Source: [ognjeeen/codex-usage-widget](https://github.com/ognjeeen/codex-usage-widget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
