## falconet

> - The shared web engine owns browser semantics: DOM projection, style, layout, paint order,

# AGENTS

## Repository boundaries

- The shared web engine owns browser semantics: DOM projection, style, layout, paint order,
  property trees, effects, hit testing, input, accessibility, and invalidation.
- Saker is the supported GUI renderer. Okojo is the supported JavaScript engine.
- Saker lowering reads resolved `FrameData` directly. Do not add another common presentation scene
  or move backend execution policy into the shared web engine.
- `Falconet.Presentation` owns compact renderer-neutral semantic contracts used during frame
  production and lowering; it is not a backend adapter or renderer boundary.

## Scripting rules

- **No Python.** Do not write or run Python scripts in this repo.
- **No `dotnet script`.** For ad-hoc tooling, test harnesses, codegen, and developer utilities, use
  a file-based C# app: one `.cs` file with `#:` directives, run with `dotnet run --file path.cs`.
- Keep file-based apps inside this repo under `tools/` or `scripts/`, not in temp dirs, and prefer
  parameters over hard-coded paths.

See `.agents/skills/csharp-scripts/SKILL.md` for the full workflow and directive reference.

## Build and test

```powershell
dotnet build
dotnet test
cargo test --manifest-path crates/Cargo.toml
```

### TUnit test filtering

Falconet and TaffySharp use [TUnit](https://github.com/thomhurst/TUnit) on Microsoft.Testing.Platform
(MTP), not VSTest. The familiar `dotnet test --filter "..."` option is rejected and runs zero
tests. Use `--treenode-filter`:

```powershell
dotnet test --treenode-filter "/*/*/LayoutAlgorithmTests/*"
dotnet test --treenode-filter "/*/*/*/BlockChildWithPreventStretch*"
```

See `TaffySharp/tests/TaffySharp.Tests/README.md` for the complete filter pattern reference.

## Verification

- For production, shared source, project, build, test, generated, or configuration changes, run
  the relevant build and tests. Run `csharpier format <path>` for changed C# files.
- Changes confined to `experiments/` need only the affected experiment project and directly
  relevant probes. Do not run the full solution for an experiment-only change.
- Documentation or repository-guidance changes do not require build/test. Verify links, search for
  stale paths and renderer claims, and run `git diff --check`.
- Determine verification from the files changed in the current task, not unrelated dirty files.

## Documentation rules

- `docs/README.md` is the map and names the authoritative architecture, reference, renderer, and
  plan documents; follow its maintenance rules.
- `docs/renderers/SAKER_RENDERER_STATUS.md` describes the current Saker implementation.
  `docs/renderers/SAKER_RENDERER_IMPLEMENTATION_PLAN.md` describes only remaining work.
- Research and implementation plans are retained when they contain reusable evidence. Label them
  as research or active plans; do not present completed phase diaries as current architecture.
- Prefer updating an existing authoritative document over adding another note. Remove a clearly
  superseded duplicate in the same change.

---
> Source: [akeit0/falconet](https://github.com/akeit0/falconet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
