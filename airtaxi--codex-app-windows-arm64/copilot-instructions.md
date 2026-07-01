## codex-app-windows-arm64

> Read `docs/build-architecture.md` before making structural changes. Keep this

# Repository Guidance

Read `docs/build-architecture.md` before making structural changes. Keep this
file as a short decision guide; architecture details belong in that document.

## Stable Contracts

- Keep `Build-CodexWoA.ps1` as the stable command-line entrypoint.
- Preserve the existing CLI parameters unless a breaking change is explicitly
  requested.
- Keep the module's public surface limited to `Invoke-CodexWoABuild` and `Resolve-CodexStorePackage`.
- Keep the seven build phases explicit and linear in `src\CodexWoA.Build\Private\Orchestration.ps1`.

## Where Changes Belong

- Source acquisition and source validation:
  `src\CodexWoA.Build\Private\Source.ps1`
- Windows runtime replacement: `src\CodexWoA.Build\Private\Runtime.Windows.ps1`
- WSL runtime replacement: `src\CodexWoA.Build\Private\Runtime.Wsl.ps1`
- ASAR handling: `src\CodexWoA.Build\Private\Asar.ps1`
- Bundled plugin handling: `src\CodexWoA.Build\Private\BundledPlugins.ps1`
- Windows sandbox handling: `src\CodexWoA.Build\Private\WindowsSandbox.ps1`
- Native build prerequisites and modules:
  `src\CodexWoA.Build\Private\VisualStudio.ps1` and
  `src\CodexWoA.Build\Private\NativeModules.ps1`
- Packaging and generated installers: `src\CodexWoA.Build\Private\Packaging.ps1`
  and `src\CodexWoA.Build\Templates\`
- Package and payload validation: `src\CodexWoA.Build\Private\Validation.ps1`
- Compatibility decisions: `src\CodexWoA.Build\Data\CompatibilityPolicy.psd1`
- Pinned build helper versions: `src\CodexWoA.Build\Data\BuildTools.psd1`
- Only genuinely shared infrastructure: `src\CodexWoA.Build\Private\Common.ps1`

## KISS And DRY

- Make the smallest domain-local change that completely solves the problem.
- Prefer explicit PowerShell over framework-like indirection.
- Do not add an abstraction for a single caller.
- Reuse an existing helper only when its responsibility genuinely matches.
- Do not move domain-specific logic into `Common.ps1` merely to shorten files.
- Centralize compatibility decisions and build tool versions in their existing data files; do not duplicate them in implementation code.
- Keep orchestration readable as a sequence of build steps. Put detailed work in the responsible domain file.

## Verification

- Add or update focused Pester tests under `tests\Unit` for behavioral changes.
- Preserve characterization tests when changing internals.
- Run `.\tests\Run-Checks.ps1` from the repository root before finishing; it is
  the required fast quality gate.
- Use `.\tests\Run-Checks.ps1 -InstallDependencies` only when required modules are unavailable.
- Do not run a full build locally unless explicitly requested. For packaging, native payload, runtime replacement, signing, or end-to-end changes, run the fast checks and report that full ARM64 validation remains for CI or an explicitly requested workflow run.

## Generated Files

Do not manually edit or commit generated content under `dist\`, `build\`, `work\`, or `.tools\`.

---
> Source: [airtaxi/codex-app-windows-arm64](https://github.com/airtaxi/codex-app-windows-arm64) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
