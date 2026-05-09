## psscriptmodule

> See [AGENTS.md](../AGENTS.md) for the full coding guidelines. This file is a focused summary for inline code generation.

# GitHub Copilot Instructions

See [AGENTS.md](../AGENTS.md) for the full coding guidelines. This file is a focused summary for inline code generation.

---

## Bias

- **Prefer**: explicit, simple, pipeline-friendly, and testable functions
- **Avoid**: aliases, complex abstractions, environment-specific assumptions, and secrets in code

---

## Design Principles

- **Fail Fast** — validate at the function boundary, throw immediately on invalid input; never let bad data propagate deeper into the call stack
- **Single Responsibility** — one function, one purpose; if a function needs an "and" to describe what it does, split it
- **Open-Closed** — extend behavior via parameter sets and new private helpers; do not modify working public APIs
- **Least Surprise** — functions must behave like built-in cmdlets: support pipeline where it makes sense, return objects not strings, respect `-WhatIf`, and write to the correct streams

---

## DO / DO NOT

### DO

- Use `[CmdletBinding()]` and `[OutputType()]` on every function
- Use approved PowerShell verbs — verify with `Get-Verb`
- Validate parameters with `[ValidateNotNullOrEmpty()]`, `[ValidateSet()]`, `[ValidateScript()]`, or `[ValidatePattern()]`
- Use `begin`/`process`/`end` blocks for pipeline-accepting functions; collect in `begin`, add in `process`, return in `end`
- Add `SupportsShouldProcess` and call `$PSCmdlet.ShouldProcess()` for destructive operations; omit for read-only functions
- Include comment-based help with `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.INPUTS`, `.OUTPUTS`, and `.EXAMPLE`
- In `catch`, log context with `Write-Verbose "$($MyInvocation.MyCommand) failed for '$Input': $_"` then `throw $_`
- Create a `.Tests.ps1` file alongside every new function

### DO NOT

- Never throw strings — always re-throw the original exception (`throw $_`)
- Never remove or bypass tests, PSScriptAnalyzer, or security scans
- Never use aliases (`Get-ChildItem`, not `gci`)
- Never write secrets, credentials, or environment-specific paths in code
- Never ignore or silently swallow errors
- Never use `Write-Host` — use `Write-Verbose`, `Write-Warning`, or `Write-Error` instead
- Never mix output types from the same function
- Never manually update `FunctionsToExport` in the module manifest — it is populated automatically by the build

---

## File Layout

| Type                    | Location                    | Naming                                                          |
|-------------------------|-----------------------------|-----------------------------------------------------------------|
| Public function + test  | `src/Public/Verb-Noun.ps1`  | `Verb-Noun` — approved verb, hyphen required                    |
| Private function + test | `src/Private/VerbNoun.ps1`  | `VerbNoun` — PascalCase, no hyphen, not exported                |
| Classes                 | `src/Classes/ClassName.ps1` | `ClassName` — PascalCase; loaded before functions by the build  |

Public functions are automatically exported by the build. Private functions must **not** follow `Verb-Noun` naming to avoid accidental export. Classes are available to all module consumers but are not listed in `FunctionsToExport`.

---

## Build & Test Commands

```powershell
Invoke-Build -Task Build              # Build module
Invoke-Build -Task Test               # All tests (unit + analysis + security)
Invoke-Build -Task UnitTests          # Pester unit tests only
Invoke-Build -Task PSScriptAnalyzer   # Static analysis
Invoke-Build -Task InjectionHunter    # Security scan
Invoke-Build -Task Export-CommandHelp # Generate help documentation
Invoke-Build -Task Clean              # Remove build output
```

---

## Testing

- Use Pester 5+
- Mock all external dependencies
- Cover parameter validation, success, and failure scenarios
- Target ≥ 70% code coverage

---

## When Uncertain

- Follow existing patterns in the repository
- Prefer private helper functions over changing public APIs
- Add tests that document assumptions

---

## Protected — Do Not Modify Without Explicit Instruction

- `.github/workflows/` — CI/CD pipelines
- `GitVersion.yml` — versioning configuration
- `tests/PSScriptAnalyzer/PSScriptAnalyzerSettings.psd1` — analysis rules

---
> Source: [WarehouseFinds/PSScriptModule](https://github.com/WarehouseFinds/PSScriptModule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
