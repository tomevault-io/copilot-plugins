## mediainfokeeper

> - Keep patches stable across Emby upgrades.

# MediaInfoKeeper Agent Guide

## Goal
- Keep patches stable across Emby upgrades.
- Prefer exact signature matching over heuristic reflection.

## Source Of Truth
- `EMBY_DOCS_DIR` selects the active method-signature snapshot under `Docs/`.
- Default: `emby_4.9.3.0`
- Signature source of truth: `Docs/${EMBY_DOCS_DIR}/`
- Primary inputs: `*_methods.txt`

## Local Dependency Layout
- DLL root directory: `/Users/honue/Documents/Emby/dlls`
- Versioned DLL directory: `/Users/honue/Documents/Emby/dlls/<emby_version>`
- Decompiled source directory: `/Users/honue/Documents/Emby/dlls/<emby_version>/source`
- Decompiled folder format: `<AssemblyName>_<version>`
- Default version suffix comes from `EMBY_DOCS_DIR`, for example `emby_4.9.3.0` -> `4.9.3.0`

## Required Research Order
1. Read `Docs/${EMBY_DOCS_DIR}/<Assembly>_methods.txt` for exact method signatures.
2. Read decompiled source under `/Users/honue/Documents/Emby/dlls/<emby_version>/source/<AssemblyName>_<version>/` for real behavior, access modifiers, inheritance, and usable entry points.
3. Use both before changing any patch that targets Emby internals.

## Missing Dependency Workflow
- Do not stop at "file missing" if the repo scripts can populate the dependency.
- If a DLL is missing, fetch it with `bash Scripts/get_emby_dll.sh <DllName.dll>`.
- If decompiled source is missing, generate it with `bash Scripts/decompile_emby_dlls.sh <DllName.dll>`.
- Pass explicit DLL names by default. Do not use the full default set unless all default DLLs are actually needed.
- Invoke scripts with `bash Scripts/...` by default. Do not assume the executable bit is present.
- `Scripts/decompile_emby_dlls.sh` only writes source folders under `/Users/honue/Documents/Emby/dlls/<emby_version>/source/`; it does not generate Markdown index files.

## macOS Decompile Notes
- `ilspycmd` may require .NET 8 even if newer runtimes are installed.
- If `ilspycmd --version` fails with missing `Microsoft.NETCore.App 8.0`, install `dotnet@8`.
- Use this environment when running decompile commands:

```bash
export DOTNET_ROOT="/opt/homebrew/opt/dotnet@8/libexec"
export PATH="$DOTNET_ROOT:$PATH"
bash Scripts/decompile_emby_dlls.sh Emby.Providers.dll
```

## Patch Rules
- Always use `PatchMethodResolver.Resolve(...)` for patch targets.
- Set explicit `ParameterTypes` whenever possible.
- Set `ReturnType` when overload ambiguity is possible.
- Use exact `BindingFlags`.
- Avoid `Predicate` unless no stable type-based signature exists.
- Do not use fallback `FindMethod(...)` name-only matching in production paths.
- When multiple overloads are valid across versions, resolve each exact overload and patch all of them.
- Keep prefix/postfix signatures compatible with every patched overload.

## Patch Workflow
1. Locate the target in `Docs/${EMBY_DOCS_DIR}/<Assembly>_methods.txt`.
2. Copy the exact parameter type order from docs.
3. Resolve dependent runtime types by full name, for example `Assembly.GetType("Namespace.Type")`.
4. Build `MethodSignatureProfile` with exact `BindingFlags`, `ParameterTypes`, and `ReturnType` when needed.
5. If a prerequisite type is missing, log `PatchLog.InitFailed(...)` with a clear reason and stop that patch.
6. Rely on existing `PatchLog.ResolveHit` and `PatchLog.ResolveFailed` for resolution outcomes.
7. Verify with `dotnet build MediaInfoKeeper.sln`.

## High-Value Decompiled Entry Points
- External subtitle scanning usually starts from:
  - `/Users/honue/Documents/Emby/dlls/4.9.3.0/source/Emby.Providers_4.9.3.0/Emby.Providers.MediaInfo/BaseTrackResolver.cs`
  - `/Users/honue/Documents/Emby/dlls/4.9.3.0/source/Emby.Providers_4.9.3.0/Emby.Providers.MediaInfo/SubtitleResolver.cs`
  - `/Users/honue/Documents/Emby/dlls/4.9.3.0/source/Emby.Providers_4.9.3.0/Emby.Providers.MediaInfo/FFProbeSubtitleInfo.cs`

## Project Map
- `Patch/`: Harmony patches and method-resolution logic
- `Patch/PatchManager.cs`: patch bootstrap and health tracking
- `ScheduledTask/`: operational tasks
- `Services/`: runtime business services
- `Configuration/` and `Options/`: plugin config models and UI wiring
- `Docs/${EMBY_DOCS_DIR}/`: signature docs
- `Scripts/`: dependency and helper scripts

## Don’ts
- Don’t use parameter count as the primary matching strategy.
- Don’t keep dead reflection helpers.
- Don’t silently swallow missing type or signature failures.
- Don’t mix unrelated refactors into patch-signature updates.

---
> Source: [honue/MediaInfoKeeper](https://github.com/honue/MediaInfoKeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
