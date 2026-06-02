## starbreaker

> StarBreaker is a Rust workspace for inspecting and exporting Star Citizen data

# StarBreaker Copilot instructions

StarBreaker is a Rust workspace for inspecting and exporting Star Citizen data
from `Data.p4k`. The workspace includes the CLI (`cli/`, binary
`starbreaker`), reusable format crates (`crates/starbreaker-*`), an MCP server
(`mcp/`), a Tauri/React app (`app/`), an Astro docs site (`website/`), and a
Blender addon (`blender_addon/`). Read `AGENTS.md` first for the full project
rules; read `blender_addon/AGENTS.md` before editing addon code.

## Required first reads

Before planning or editing, read the relevant instruction files in this order:

1. `AGENTS.md` in the StarBreaker root.
2. `blender_addon/AGENTS.md` before any Blender addon work.
3. This file (`StarBreaker/.github/copilot-instructions.md`).

Do not rely on memory when these files are available. Re-read them when changing
component, starting a new phase, or resuming after a long interruption.

## Context: This repo

- StarBreaker is the main git repository; work inside this folder.
- The workspace root is at `../../` and contains workspace-level guidance, research
  documentation, and generated/exported packages. Follow workspace root `AGENTS.md`
  for broader context, but this file takes precedence for StarBreaker-specific work.

## Build, test, and lint commands

Run commands from the StarBreaker git root, not the parent `scorg_tools/`
workspace.

```bash
# Rust iteration build; dev profile has opt-level=1 and optimized deps.
cargo build

# Release CLI build, used for final exporter runs and CI artifacts.
cargo build --release -p starbreaker

# Full Rust workspace tests.
cargo test --workspace

# CI-equivalent Rust checks.
cargo build --release --workspace
cargo test --release --workspace

# Target one crate or one test by substring.
cargo test -p starbreaker-3d --lib
cargo test -p starbreaker-3d validate_lights_extraction -- --nocapture

# Lint/format Rust when the components are installed.
cargo clippy --workspace --all-targets
cargo fmt --all
```

```bash
# Blender addon tests use stubbed bpy and run on system Python.
cd blender_addon
python3 -m unittest discover -s tests -q
python3 -m unittest discover -s tests -p 'test_manifest.py' -q
```

```bash
# Tauri app frontend.
cd app
npm ci
npm run build
npm run lint
npm run tauri build

# Documentation website.
cd website
npm ci
npm run build
```

For native decomposed `.blend` export performance/regression work, use the
checked-in profiler:

```bash
tools/profile_aurora_blend.sh /tmp/starbreaker_aurora_profile 0
```

The CLI auto-detects common Star Citizen install paths. Set `SC_DATA_P4K` only
when targeting a non-default install. The canonical Aurora decomposed export is:

```bash
SC_DATA_P4K="$HOME/Games/star-citizen/drive_c/Program Files/Roberts Space Industries/StarCitizen/LIVE/Data.p4k" \
  target/release/starbreaker entity export "aurora_mk2" \
  "../ships" --kind decomposed --format blend --lod 0 --mip 0 --materials all
```

## Architecture

- `crates/starbreaker-3d` owns entity export. The pipeline is split under
  `src/pipeline/`: `loadout.rs` resolves DataCore loadouts and item ports,
  `interiors.rs` handles interior placements, `textures.rs` and `palette.rs`
  build shared package assets, `glb_assembly.rs` assembles GLB output, and
  `blend_assembly.rs` writes native `.blend` package scenes and decomposed
  asset libraries. `decomposed.rs` coordinates package output and shared
  `Data/...` asset reuse.
- Lower-level format crates are deliberately separate: `starbreaker-p4k` reads
  archives, `starbreaker-datacore` parses the database, `starbreaker-cryxml`
  decodes binary XML, `starbreaker-chunks` handles CryEngine chunk files,
  `starbreaker-dds` decodes textures, and Wwise/WEM/CHF crates cover audio and
  character formats.
- The decomposed export contract is shared between Rust and Blender. Rust emits
  `Packages/<package>/scene.json`, `palettes.json`, `liveries.json`, material
  sidecars, textures, and reusable assets under `Data/...`; the Blender addon
  imports those packages and reconstructs materials, lights, palette/livery
  state, and animation controls.
- `blender_addon/starbreaker_addon/runtime/importer/` composes
  `PackageImporter` from mixins: palette, decals, layers, materials, builders,
  groups, and orchestration. Keep new per-entity import behavior in focused
  helpers or mixins rather than growing orchestration.
- The Tauri app wraps the Rust crates from `app/src-tauri` and uses the React
  frontend in `app/src`. The website is independent Astro/Starlight content in
  `website/`.

## MCP workflows

- StarBreaker MCP is the preferred way to inspect game data while researching
  exporter bugs. Use it for fast DataCore, P4k, material, texture, chunk, and
  animation lookups instead of writing one-off CLI probes.
- Use `search_entities` for EntityClassDefinition lookup, `search_records` for
  all DataCore record types, `datacore_record` for full JSON records, and
  `datacore_query` for a specific property path. For raw loadout data, query
  `Components[SEntityComponentDefaultLoadoutParams]`; `entity_loadout` returns
  StarBreaker's processed/resolved tree.
- Use `p4k_search`, `p4k_list`, and `p4k_read` to find and inspect archive
  paths. Use `mtl_summary` for material XML summaries, `image_preview` for
  DDS/PNG/JPG visual checks, `chunk_list`/`chunk_read` for CryEngine chunk
  structure, `dba_dump` for CAF/DBA animation channels, and `mannequin_dump` for
  fragment/scope metadata.
- Use the CLI, not MCP, for full export operations and end-to-end pipeline
  validation, e.g. `starbreaker entity export ...`.
- If the StarBreaker MCP server code changes, rebuild `starbreaker-mcp` and
  replace the deployed binary under `mcp/` after stopping the running MCP
  process. The client must be restarted to pick up the new server binary.
- On systems with Blender MCP, use it for generated `.blend` validation:
  inspect object hierarchy, transforms, linked libraries, missing files, world
  nodes, screenshots, and test renders directly in Blender. Use background
  Blender MCP file-summary tools for closed files, and execute Python in the
  connected Blender instance for live-scene checks. For native `.blend`
  transform, light, hierarchy, or render fixes, validate the generated file in
  Blender rather than relying only on JSON or Rust-side assertions.
- When driving Blender through MCP, reset scenes with
  `bpy.ops.wm.read_homefile(app_template="")`. Do not hand-delete objects,
  node groups, materials, or orphaned data. After addon edits, sync the live
  addon install before validating:

  ```bash
  rsync -a --delete blender_addon/starbreaker_addon/ \
    ~/.config/blender/5.1/scripts/addons/starbreaker_addon/
  ```

  Then purge cached `starbreaker_addon` modules, reset the scene, and re-enable
  the addon before validating.

## Mandatory troubleshooting principles

- Fix root causes, not symptoms.
- Do not add fallback defaults, clamps, broad catches, silent early returns, or
  placeholder data unless the surrounding code already has a documented,
  intentional pattern for that exact behavior.
- **No heuristics or hard-coded asset fixes in production code.**
  - Do not gate code on a specific ship, object name, mesh name, material name,
    texture path, asset path, item ID, or instance suffix.
  - Do not use branches like `if name == ...` or `matches!(asset_path, ...)` for
    exporter behavior.
  - Named assets are acceptable ONLY in regression tests, diagnostics, temporary
    repro scripts, and validation notes.
  - A production fix must be derived from structural source data such as NMC node
    metadata, transform basis, geometry flags, shader family, blend mode,
    material XML, `.chrparams`, DBA/CAF metadata, DataCore records, or Blender
    SDNA/API behavior.
  - If the structural rule is not known, investigate it with StarBreaker MCP,
    Blender MCP, source data, and tests. Do not commit a name-matched workaround
    as a temporary fix.

## Key conventions

- New Rust source files must start with a `//!` module-doc header describing the
  file responsibility, important public types, and key functions. Update the
  header when the file's role changes.
- Use debug builds for normal Rust iteration. Release builds are for CI,
  deployable binaries, MCP server deployment, and final/export benchmark runs.
- The CLI package and binary are named `starbreaker`; do not refer to a
  `starbreaker-cli` package.
- For decomposed exports, the output argument is the shared export root that
  contains `Packages/` and `Data/`, not an individual package folder. Passing a
  package directory causes nested `Packages/.../Packages/...` output.
- Keep export contract changes backward-compatible and update
  `docs/decomposed-export-contract.md` when adding fields. Material naming and
  shader-family inputs are documented in
  `docs/blender-material-contract-naming-rules.md` and related docs.
- Do not run multiple cargo builds/tests in parallel against the same target
  directory; agents sharing the same build output can race.

## Visual Fix Loop (VFL)

### Purpose

Use VFL for any task where correctness is visual, interactive, subjective, or
hard to prove with programmatic tests alone.

VFL applies to:

- Blender visuals: lighting, UVs, materials, decals, transforms, render output,
  linked libraries, view settings, hierarchy, and scene assembly.
- Blender addon UI/UX and import behavior.
- Tauri UI/UX.
- Performance where the user-visible result cannot be fully measured by an
  existing automated test.

The goal is to validate fixes in Blender through MCP whenever possible before
spending time on Rust rebuild, re-export, and user-verification cycles.

### Entry conditions

Enter VFL when any one of these is true:

- The output must be visually verified.
- The result is unclear, subjective, or depends on user perception.
- A fix may require rebuild/re-export/reload to validate.
- The user is unsure whether a previous fix worked.
- The task affects Blender scene appearance, addon behavior, Tauri UI behavior,
  or perceived performance.

When VFL applies, explicitly follow the VFL loop. Do not silently switch to a
normal code-only workflow.

**MANDATORY**: If ANY of the entry conditions are met, VFL is **required**, not
optional. Implementing code changes without VFL validation for Blender-visible
fixes is a protocol violation. Always enter VFL and stay in the loop until the
user confirms the fix or instructs you to stop.

### Core rules

1. **MCP first**
   - Blender MCP is the only live validation path.
   - If a fix can be tested or approximated in the current Blender instance,
     test or prototype it there before modifying Rust.
   - For transform/material/UI issues that can be manipulated live (for example:
     rotating empties/objects, adjusting node inputs, trying alternate matrix
     application), first prove the candidate behavior in Blender MCP and ask the
     user to confirm before writing persistent exporter/addon code.
   - Do not trigger Rust rebuilds or full re-exports until they are necessary to
     make the fix persistent or to test behavior that cannot be exercised live.

2. **Strict loop**
   Repeat these steps until the user confirms success or tells you to stop:
   - Research the issue. Use sub-agents when the investigation can be safely
     parallelized.
   - Write or update the plan.
   - Work one scoped item at a time (do not batch a full multi-item implementation).
   - Implement one focused fix for the current item only.
   - Test in the current Blender instance with MCP if possible.
   - If MCP-only validation is impossible, rebuild/re-export/reload and then
     test.
   - Ask the user to verify that single item using the question tool before moving
     to the next item.

3. **User verification required**
   - Every attempted visual or interactive fix is a hypothesis.
   - Never claim a VFL fix is complete until the user confirms it.
   - After every attempt, ask the user to verify using the question tool.

4. **Question tool rules (MANDATORY)**
   - **ALWAYS use the question tool** for VFL verification. Never ask in plain text.
   - Ask one clear question.
   - Include a clear success choice, such as `Yes, this is fixed`.
   - **Always provide a free-text input option** so the user can explain what is
     still wrong or partially working.
   - If multiple issues are in scope, the question must let the user confirm
     that all scoped issues are fixed or describe which issue remains.

5. **Assume failure until confirmed**
   - Treat every fix as unconfirmed until the user says it worked.
   - If the user reports failure or partial success, continue the VFL loop.
   - Do not commit VFL fixes until the scoped issues are confirmed fixed unless
     the user explicitly instructs you to commit an intermediate state.

### Rust and Tauri workflow inside VFL

Use this sequence when a change cannot be validated only through MCP:

1. Prototype or approximate the behavior in Blender MCP if possible.
2. Implement the persistent fix in Rust, Python, or Tauri code.
3. Run the smallest relevant build/test command.
4. Rebuild/re-export/reload only what is required.
5. Validate in Blender MCP or the relevant UI.
6. Ask the user for confirmation using the question tool.
7. Repeat until confirmed.

### Performance inside VFL

If VFL work makes load, refresh, import, render setup, or UI interaction slower:

- Treat the slowdown as part of the same task.
- Investigate and fix it before exiting VFL.
- Prefer existing profiling tools. If a profiler was previously created and
  removed, recreate it and keep it for future use.
- Preserve behavior while optimizing. For exported `.blend` assets, compare
  selected MD5 sums before and after performance changes when the task requires
  binary stability.

### Crash handling

If Blender crashes during MCP use:

1. Stop the current validation attempt.
2. Identify the likely cause from recent changes or the last MCP operation.
3. Add a prevention fix or reduce the repro to a stable operation.
4. Resume testing only after the crash cause is addressed or isolated.

If Blender must be restarted:

- Ask the user using the question tool to restart Blender.
- Wait for confirmation before continuing MCP validation.
- Do not ignore Blender crashes or treat them as unrelated unless proven.

### Scope control

- Stay focused on the current VFL plan.
- If the user reports a new issue that is a regression from current changes, fix
  it inside the same VFL loop.
- If the user reports an unrelated issue, record it in the plan or task notes,
  then ask using the question tool whether it should be added to the current
  plan.
- Continue the current task unless the user explicitly changes priority.

### Exit conditions

Exit VFL only when one of these is true:

- The user explicitly confirms all scoped issues are fixed.
- The user tells you to stop, pause, commit an intermediate state, or move on.

### Key principle

Blender MCP is the fastest feedback loop for Blender-visible behavior. Use it
whenever possible. Rebuild only when required.

## Planning and sub-agents

- Use sub-agents when work decomposes into independent research, implementation,
  or review tracks.
- Do not run parallel agents that will race on the same Cargo target directory,
  generated export root, Blender instance, or mutable file set.
- Give sub-agents complete context, including the no-hardcoding rule, relevant
  file paths, validation target, and required tests.
- Require sub-agents doing code work to self-review for architecture,
  integration, tests, error handling, conventions, regressions, performance, and
  documentation before reporting complete.
- After sub-agent work, perform a manager review before accepting the result.

## Asking questions about planning and next steps

When asking the user about planning decisions or what to do next (e.g., which
of multiple options to pursue, whether to move to a different task, whether to
prioritize a discovered issue), **ALWAYS use the question tool**.

Provide:
- A clear question.
- Relevant options as choices (if multiple discrete options exist).
- A free-text input option so the user can explain or suggest alternatives.

Never ask planning questions in plain text output.

---
> Source: [diogotr7/StarBreaker](https://github.com/diogotr7/StarBreaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
