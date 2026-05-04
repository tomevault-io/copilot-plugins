## gadgetexplorer

> `GadgetExplorer` is a .NET CLI for finding serializer-driven gadget chains that can reach configured sinks.

# AGENTS

## What This Repo Is

`GadgetExplorer` is a .NET CLI for finding serializer-driven gadget chains that can reach configured sinks.
It loads assemblies with `dnlib`, builds a call/dispatch graph from IL, applies serializer-specific trigger
rules, and reports trigger-to-sink paths in text or JSON.

## Repo Map

- `src/GadgetExplorer/Application`: CLI parsing, scan runner, progress, text/JSON report writers.
- `src/GadgetExplorer/Analysis/Loading`: input expansion, `runtimeconfig` inspection, assembly resolution,
  module loading.
- `src/GadgetExplorer/Analysis/Index`: type/method/property/event indexing and graph construction.
- `src/GadgetExplorer/Analysis/SinkAnalyzer*`: root filtering, sink resolution, reverse slicing, finding output.
- `src/GadgetExplorer/Configuration`: sink/profile JSON loading and shipped-resource path resolution.
- `src/GadgetExplorer/resources`: shipped profiles, sinks, and ignore-sinks.
- `samples`: small fixture projects used by tests.
  `SmokeSample` is the main analysis corpus; `MessagePackRuntimeSample` and
  `InterfaceStrictContractSample` cover narrower behavior.
- `tests/GadgetExplorer.Tests`: unit and behavior tests.
- `docs/Serializer-Behavior-Matrix.md`: serializer behavior reference.
- `.github/workflows/dotnet.yml`: CI restores and runs the test project on .NET 9.

## How It Works

The main pipeline is:

1. `Program.cs` parses CLI options and calls `ScanRunner`.
2. `AssemblyInputLoader` expands file/directory inputs, inspects `runtimeconfig` files, builds a load plan,
   and loads reachable modules.
3. `AnalysisIndex.Build(...)` indexes types/methods/properties/events, builds dispatch maps, builds per-method
   control-flow graphs, simulates stack/locals/fields, and emits graph edges for calls, constructors, delegate
   relays, async/iterator entry points, and event raises.
4. `SinkAnalyzer` resolves configured sinks, computes eligible root classes from the serializer profile, performs
   reverse sink slicing, and emits trigger-to-sink findings.
5. Reporting writes either the text report (`ScanReportWriter`) or JSON (`ScanJsonReportWriter`).

## Profiles, Sinks, And Options

- Shipped profiles live in `src/GadgetExplorer/resources/serializer-profiles`.
  Current built-ins are `BinaryFormatter`, `JsonDotNet`, `JsonDotNetGetters`, `MessagePackTypeless`,
  `PublicTwoStringConstructor`, and `XmlSerializer`.
- Profiles describe root eligibility, trigger policy, activation policies, callbacks, and custom
  deserialization methods.
- Shipped sink packs live in `src/GadgetExplorer/resources/sinks` and are grouped by family
  (`code-execution`, `filesystem`, `ssrf`, `xxe`, etc.).
- The default ignore pack lives in `src/GadgetExplorer/resources/ignore-sinks` and suppresses known framework
  noise.
- Sink config matching can be broad or exact:
  omitting parameters means "match overload family";
  `parameters: []` means "match the parameterless overload exactly".
- Important CLI options live in `ScanCommandLineParser` / `ScanOptionValues`:
  `--profile` / `--profile-file`, `--sinks`, `--ignore-sinks`, `--interface-expansion`, `--sort`,
  `--max-path-length`, `--assembly-resolution-mode`, `--output`, `--output-format`.
- Interface expansion modes: `off`, `strict`, `broad`.
- Assembly resolution modes: `restricted`, `inference-no-fallback`, `inference-with-fallback`.
- Output formats: `text`, `json`.

## Running

- Build: `dotnet build .\GadgetExplorer.sln`
- Test: `dotnet test .\tests\GadgetExplorer.Tests\GadgetExplorer.Tests.csproj`
- Artifacts go under `artifacts\bin\...` because `Directory.Build.props` enables artifacts output.
- Run the built tool from `artifacts\bin\GadgetExplorer\debug\GadgetExplorer.exe` or `release`.
- Shipped `sinks`, `ignore-sinks`, and `serializer-profiles` are copied beside the executable and resolved from
  `AppContext.BaseDirectory`.
- `samples/SmokeSample/sample-sinks.json` is a small custom sink example.

## Working Rules

- Keep changes small and follow existing patterns/style.
- Do not change major analysis behavior unless the task explicitly calls for it.
- Prefer updating the narrowest layer that owns the behavior.
- Start each separate piece of work from a clean branch based on `origin/main`.
- Do not stack a new PR on top of a previous topic branch; fetch, switch, and branch again from the latest `origin/main`.
- Keep work in the `GadgetExplorer` repo/worktree you intend to ship from, and verify `git rev-parse --show-toplevel` before editing when multiple local clones exist.
- When a branch is meant for GitHub review, wait for its server-side checks to finish before starting the next PR branch if the outcome could affect the follow-up work.
- If you change loading, indexing, dispatch, sink matching, profiles, or reporting, add or update tests near the
  affected area.
- Validate with targeted tests first, then run the relevant broader test command before finishing.

---
> Source: [nines-nine/GadgetExplorer](https://github.com/nines-nine/GadgetExplorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
