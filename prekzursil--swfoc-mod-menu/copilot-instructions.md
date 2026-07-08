## swfoc-mod-menu

> Every line of code must pass all quality gates on first push. Zero issues across all providers. 100% line and branch coverage. No suppressions, no exclusions, no workarounds.

# AGENTS.md — Universal Quality Contract for Codex

## Hard Requirements

Every line of code must pass all quality gates on first push. Zero issues across all providers. 100% line and branch coverage. No suppressions, no exclusions, no workarounds.

## Before Writing Code

1. Read gate configs: `.github/workflows/`, `.codacy.yml`, `sonar-project.properties`, `.deepsource.toml`
2. Plan tests first — write test signatures before implementation
3. Check complexity budget — will any function exceed CCN 15? Split before writing

## Coding Constraints

### C# / .NET
- `<Nullable>enable</Nullable>` in every .csproj — fix all CS8600-CS8604 warnings
- `ArgumentNullException.ThrowIfNull()` for every reference parameter
- `?.` and `??` for nullable member access — never use `!` null-forgiving
- `[assembly: CLSCompliant(true)]` — no `uint`/`sbyte`/`ushort` in public APIs
- Catch specific exceptions only — never bare `catch (Exception)`
- CCN ≤ 15, function length ≤ 50 lines, parameters ≤ 5
- `dotnet build --warnaserror` must pass with 0 warnings

### Python
- Type annotations on all function signatures
- No `shell=True` in subprocess calls
- No hardcoded secrets — use `os.environ[]`
- No `eval()`/`exec()` with untrusted input
- CCN ≤ 15, function ≤ 50 lines, parameters ≤ 5
- `bandit -r src/ -ll` must show 0 findings

### TypeScript
- `"strict": true` — no `any` types
- ESLint `--max-warnings 0`
- No `console.log` in production code

### C++
- Smart pointers over raw new/delete
- `static_cast` over C-style casts
- `constexpr` where possible
- Lowercase includes, include what you use

## Forbidden Practices

- `// NOSONAR`, `#pragma warning disable`, `# noqa`, `// codacy:ignore`
- `[SuppressMessage]` attributes
- `.codacy.yml` file exclusions on first-party code
- Lowering any threshold or gate
- Marking issues as "false positive" without human approval
- Pushing without running local verification

## Pre-Push Verification (MANDATORY)

Run the appropriate verification commands and confirm 0 failures before every push. If verification fails, fix before pushing. Never push failing code.

## Coverage

100% line AND branch. Every public method tested. Every branch tested. Every new line covered. Coverage reported to ALL providers: Codecov, SonarCloud, Codacy, QLTY, DeepSource.

## Definition of Done

1. Tests pass (verified by running, not assumption)
2. Coverage 100% (verified by report)
3. Local analysis 0 issues (verified by tools)
4. CI green (verified by GitHub Actions)
5. All dashboards 0 issues (verified by visiting URLs)
6. No suppressions used

---
> Source: [Prekzursil/SWFOC-Mod-Menu](https://github.com/Prekzursil/SWFOC-Mod-Menu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
