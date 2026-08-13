## nuri

> - This codebase builds a renderer on NVRHI. Keep NVRHI types private and make higher-level code depend on Nuri's typed descriptors, handles, and submission contracts.

# AGENTS

## Project intent
- This codebase builds a renderer on NVRHI. Keep NVRHI types private and make higher-level code depend on Nuri's typed descriptors, handles, and submission contracts.
- Correctness is the top priority, with performance as a critical constraint to be optimized after correctness is ensured. Optimize throughput and frame time only once behavior is correct.

## C++ and memory
- Use C++20 and the STL for core functionality.
- Use `std::string_view` for immutable strings (debug names, string literals, error/debug/info messages etc.)
- Use `std::span` for non-owning views on containers
- Prefer PMR types and custom allocators to control allocations and reduce churn.
- For transient allocations with clear scope lifetime (loop iteration/task/frame slice), prefer `ScratchArena` + `ScopedScratch` from `lib/nuri/core/pmr_scratch.h`.
- Do not let scratch-backed allocations escape the scope of the `ScopedScratch` object; all scratch-backed objects must be destroyed before the `ScopedScratch` exits.
- Do not nest `ScopedScratch` over the same `ScratchArena`.
- Avoid exceptions when possible; use `lib/nuri/result.h` for error handling.

## Renderer engineering
- For renderer architecture, semantic compression, N+1 design, RHI seams,
  PIMPL decisions, GPU lifetime, and measured performance work, follow
  `.codex/skills/nuri-renderer-design/SKILL.md` and its focused references.

## Local documentation and scratch work
- `docs/` and `.scratch/` are intentionally Git-ignored but remain part of the repository's local agent context. Read and update relevant files there when a task calls for them; ignored does not mean disposable.
- Ignore-aware discovery may omit these directories. Use `rg --files --no-ignore docs .scratch` when listing their contents.

## Task workflow dispatch
- Classify every user request before acting.
- Handle simple questions, brief explanations, status checks, and ordinary conversation directly. Do not create a scratchpad or spawn subagents for them.
- For substantive research, design, diagnosis, performance investigation, or feature implementation with code, shader, build, or manifest changes, follow `.codex/skills/nuri-task-workflow/SKILL.md`.
- If a simple request grows into multi-step work, switch to the substantive workflow at that point.

## Tests and renderer validation
- Be exceptionally careful when adding unit tests. Add one only for a critical invariant that the test covers completely and deterministically at the correct seam; partial or incidental coverage is not sufficient.
- Before adding a unit test, document why an autotest, visual snapshot, or benchmark cannot cover the failure more directly.
- Prefer renderer autotests for multi-frame behavior and readouts, visual snapshots for pixels and rendering semantics, and benchmarks for performance.
- Do not add unit tests that merely mirror pass registration, draw layout, implementation structure, visual output, or performance behavior already owned by those renderer tools.
- Keep the remaining unit-test layer selective: pure algorithms, lifecycle and ownership contracts, render-graph hazards, handles and schemas, cache behavior, and similarly important deterministic invariants.

## Shaders
- Shaders are written in GLSL.

## Build and scripts
- Do not run `cmake` manually: it can overwrite generated files, bypass project configuration, and produce inconsistent builds; use the build scripts in `scripts/` instead.
- CMake is the build system.
- Build/run scripts live in `scripts/`.

## Profiling (Tracy)
- For renderer performance work, use `.codex/skills/nuri-benchmarks` and route diagnostic Tracy traces through `nuri-bench --tracy-diagnostic` so benchmark reports own both metric JSON and trace artifacts.
- Enable CPU profiling through the build scripts, not direct CMake: pass `cpu` to the relevant `scripts/build_*` or `scripts/run_*` wrapper. Debug builds enable Tracy by default; Release builds need the explicit mode.
- Instrument code using `lib/nuri/core/profiling.h` macros:
  - `NURI_PROFILER_FUNCTION()` / `NURI_PROFILER_FUNCTION_COLOR(color)`
  - `NURI_PROFILER_ZONE(name, color)` / `NURI_PROFILER_ZONE_END()`
  - `NURI_PROFILER_FRAME(name)` and `NURI_PROFILER_THREAD(name)`
- Focus zones on likely bottlenecks (frame loop, render submission, shader compilation, asset loading). Keep zones coarse and meaningful; avoid spamming per-draw zones unless you’re drilling down.

## Logging
- `fatal`: use for paths that will crash or abort the app.
- `error`: use for paths that return a recoverable error that can be handled by callers.
- `warning`: use for paths where behavior is unexpected but not critical to application function.
- `info`: use for generic app behavior and user-relevant runtime information (device info, startup phases, etc.).
- `debug`: use in non-hot paths to trace creation/initialization of resources and systems.

## Naming conventions
- `ModelData` is CPU-side asset data; `Model` is the GPU/renderable object.
- `Buffer` and `Texture` refer to GPU resources, not raw data containers.

---
> Source: [Panbok/nuri](https://github.com/Panbok/nuri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
