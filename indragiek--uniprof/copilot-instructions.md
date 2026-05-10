## uniprof

> Uniprof is a universal profiling tool that wraps platform profilers (py-spy, 0x, rbspy, async-profiler, dotnet-trace, perf, xctrace, etc.) to provide a consistent, cross‑language experience. It prefers Docker to minimize setup and ensure reproducible results.

# Uniprof Codebase Guide

Uniprof is a universal profiling tool that wraps platform profilers (py-spy, 0x, rbspy, async-profiler, dotnet-trace, perf, xctrace, etc.) to provide a consistent, cross‑language experience. It prefers Docker to minimize setup and ensure reproducible results.

## Project Layout

```
uniprof/
├── src/
│   ├── index.ts            # CLI entry
│   ├── version.ts          # Single source of truth for version
│   ├── commands/           # CLI command implementations
│   ├── mcp/                # Model Context Protocol server + tools
│   ├── platforms/          # Language/platform plugins
│   ├── types/              # Type definitions
│   └── utils/              # Shared utilities
├── containers/             # Docker images per platform
├── speedscope/             # Bundled Speedscope viewer
├── tests/                  # Test suite
└── docs/                   # Documentation
```

## Quick Start

- Simplest: profile and analyze in one step
  - `uniprof python app.py`
  - `uniprof --visualize node server.js`
- Save then analyze/visualize later
  - `uniprof record -o profile.json -- ./my-app`
  - `uniprof analyze profile.json`
  - `uniprof visualize profile.json`

## CLI Reference

### Aliases & Parsing
- `uniprof <cmd>` ≈ `uniprof record --analyze -- <cmd>` (unless `--visualize`, which maps to `record --visualize`).
- Options before the first non‑option belong to uniprof; after, they belong to your command.
- `record` inserts `--` automatically when omitted (so trailing args reach your app).
- `--extra-profiler-args` accepts dashed values directly or as a single quoted string; the alias layer condenses them.

Examples
```
uniprof python app.py
uniprof -- python app.py
uniprof --visualize -- node server.js
uniprof -o out.json --analyze -- ./my-app
```

### Commands
- `bootstrap` — Environment checks and setup guidance
- `record` — Record a profile
- `analyze` — Analyze an existing profile
- `visualize` — Serve the profile in a local Speedscope
- `mcp` — Run or install the MCP server

Advanced options
```
uniprof record --mode host -o profile.json -- python app.py
uniprof record -o profile.json --extra-profiler-args --rate 500 -- python app.py
uniprof analyze profile.json --threshold 5 --filter-regex "MyApp\." --min-samples 100 --max-depth 10
```

### Path Validation (Container Mode)
- Absolute argument paths outside the working directory cause errors; they won’t be mounted.
- Option values that look like paths outside the working directory trigger warnings.
- Host mode skips these checks.

## Platforms

### Supported & Defaults
- Sampling defaults (≈999Hz) unless platform‑fixed:
  - Python/Ruby: `--rate 999`
  - PHP: `--period 0.001001001` (≈999 Hz)
  - JVM: `--interval 1001001ns`
  - perf/BEAM: `-F 999`
  - Node.js (0x), xctrace: fixed by runtime/system
- Users can override via `--extra-profiler-args`.

### macOS `.app` Bundles (xctrace)
- Resolves app bundles to CFBundleExecutable under `Contents/MacOS` and validates permissions.

## Containers & Host

- Docker is preferred; host mode is available per platform.
- Host networking: supported on Linux; on macOS requires Docker Desktop host‑networking support (see Docker settings/diagnostics). When requested and not available, a warning is shown.
- Symbol resolution with perf in containers uses `--symfs /` and build‑id caching; some paths may still appear unknown.

## Analysis

### Profile Types
- Sampled: py-spy, rbspy, Excimer, async-profiler (in sampled mode), 0x, perf
  - Statistical samples with sample counts correlated to CPU time
- Evented: dotnet-trace (EventPipe), Instruments
  - Entry/exit events converted to synthetic weighted samples for consistent analysis

### Analyze Options
- `--threshold <percent>`: Minimum percent of total time to display (default 0.1)
- `--filter-regex <pattern>`: Filter by function or file path
- `--min-samples <count>`: Minimum sample count to show
- `--max-depth <depth>`: Consider leaf‑most N frames per sample

Implementation details
- Evented → sampled conversion attributes elapsed time to the current stack; computes totals, self time, and percentiles when weights vary.
- .NET: Uses EventPipe `cpu-sampling` with precise timing and full call stacks; use `--platform dotnet` if auto‑detect fails.

## Visualization

### Speedscope Viewer
- Build copies `speedscope/` to `dist/speedscope`.
- At runtime, serve from `dist/speedscope` or fallback to repo `speedscope/`.
- If not found, run `npm run build`.

## MCP Integration

Commands
```
uniprof mcp run
uniprof mcp install <client>
```

Tool: `run_profiler`
- Required: `command`, `cwd`
- Optional: `platform`, `mode`, `output_path`, `enable_host_networking`, `extra_profiler_args`, `verbose`
- Returns analyzed results (pretty or JSON)

Schema notes
- The MCP tool registers a Zod schema shape (not wrapped in `z.object(...)`). Convert to JSON Schema at client integration boundaries when needed; do not change the shape in `src/mcp/tools.ts`.
- Zod import path is intentionally `zod/v3` in MCP files to align with SDK expectations; do not change this import.
- MCP runtime reuse: the tool re‑invokes the current entry using the same runtime (`process.argv[0]`).

## Plugin Architecture

### Key Interfaces
- `PlatformPlugin`: detection, env checks, container/host execution, post‑processing, analysis formatting, examples, options, sampling rate, etc.
- `ProfileContext`: typed state across the profiling lifecycle.
- Docker integration: containers handle privileged ops; bootstrap scripts set up the environment; cache volumes persist dependencies.

### Contract
- `runProfilerInContainer(args, output, options, ctx)`
  - Perform the run; set `ctx.rawArtifact` to the raw output and register temp paths.
- `buildLocalProfilerCommand(args, output, options, ctx)`
  - Build argv for host mode; if a raw file is produced (perfdata/nettrace/ticks/etc.), call `setRawArtifact`.
  - Inject env via `mergeRuntimeEnv` when needed.
- `postProcessProfile(raw, finalOutput, ctx)`
  - Convert `ctx.rawArtifact` → Speedscope JSON; set exporter; write `finalOutput`; cleanup temps.

### ProfileContext Helpers (`src/utils/profile-context.ts`)
- `setRawArtifact(ctx, type, path)`
- `mergeRuntimeEnv(ctx, env)`
- `addTempFile/Dir(ctx, path)`
- `finalizeProfile(ctx, inputPath, finalPath, exporter)` sets exporter + moves file + cleanup

## Subprocess & Signals

- Use `src/utils/spawn.ts` (execa) for processes; prefer async `spawn(...)` and `readAll(...)`.
- Ctrl+C behavior:
  - First SIGINT: try to signal only the profiled application (child processes). If not found, fallback to process group (host) or PID 1 (container).
  - Second SIGINT in a short window: terminate the profiler and exit.

## Development

### Module System
- ESM‑only; no `require(...)` in `src/`.
- Use static imports top‑level; dynamic `await import(...)` inside async functions when necessary.

### Version & Commands
- Version: `src/version.ts` (used by CLI, MCP, tests).
- Scripts:
  - `npm run build` — compile + copy speedscope
  - `npm run dev` / `npm start` — run compiled CLI
  - `npm run typecheck` — TS typecheck
  - `npm run lint` / `lint:check` — format/lint (Biome)
  - `npm test` — run tests

### Code Style
- Prefer “why” comments over “what”.
- JSDoc for public APIs; clarify non‑obvious logic.
- TypeScript strict; composition over inheritance where possible; graceful error handling.

## Additional Notes & Intentional Behaviors

- perf container symbol resolution: best‑effort with `--symfs /` and build‑id caching.
- 0x ticks conversion: unit `milliseconds`, weight `1` per sample.
- Node (0x) in container: prefers the latest nvm‑managed Node; if missing, prints guidance to bootstrap/rebuild image.
- Visualize server: serves from `dist/speedscope` or repo `speedscope/` with safe path checks and correct MIME types.
- Record alias parsing:
  - Single source of truth for `record` options in `src/utils/cli-parsing.ts` via `RECORD_BOOLEAN_OPTS` / `RECORD_VALUE_OPTS`.
  - When changing `src/index.ts` options, update these sets to keep alias behavior in sync.
- `.NET SDK 10` supports `dotnet run <file.cs>`; do not normalize that away.
- `perf -k mono` is intentional for JIT‑heavy cases (e.g., BEAM).
- Commander’s built‑in `help` is used; there’s no custom `help` command.

---
> Source: [indragiek/uniprof](https://github.com/indragiek/uniprof) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
