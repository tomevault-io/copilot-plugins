## ovrtx

> This file gives AI coding agents the minimum context needed to work effectively in this repository. Use it as a starting map, then go to `skills/` for task-level implementation guidance.

# AGENTS.md - AI Agent Guide for ovrtx

This file gives AI coding agents the minimum context needed to work effectively in this repository. Use it as a starting map, then go to `skills/` for task-level implementation guidance.

## What This Repo Is

`ovrtx` is a pre-release NVIDIA SDK that exposes Omniverse RTX through:
- a Python package (`python/ovrtx/`)
- a C API (documented in `docs/c_api/`)
- runnable examples in both Python and C (`examples/`)

Primary use case: OpenUSD-based sensor simulation and rendering workflows (camera, lidar, radar, etc.).

## Start Here

- Read `README.md` for top-level product context and quick starts.
- Read `examples/README.md` to choose a runnable reference project.
- Read `skills/README.md` to understand the skill format and maintenance expectations.

## Repo Layout (High-Level)

- `python/ovrtx/` - Python package source
- `tests/` - Python test suite (pytest)
- `examples/python/` - Python example projects (`minimal`, `planet-system`)
- `examples/c/` - C/C++ example projects (`minimal`, `vulkan-interop`)
- `skills/` - Task-oriented agent skills (`*/SKILLS.md`)
- `docs/` - Sphinx docs, including Python/C getting started and API reference scaffolding

## Common Workflows

### Python (recommended via uv)

- Use Python 3.10-3.13.
- Run example:
  - `cd examples/python/minimal`
  - `uv run main.py`

### C/C++ (CMake)

- Build example:
  - `cd examples/c/minimal`
  - `cmake -B build`
  - `cmake --build build --config Release`
- Run binary on Linux:
  - `./build/minimal`

### Tests

- Python tests live under `tests/`.
- If working on Python bindings/API behavior, run targeted tests first, then broader suites as needed.

## Use Skills for Task-Specific Work

When a request maps to a known ovrtx workflow, go directly to the relevant skill in `skills/`:

- Renderer setup/init -> `skills/renderer-creation/SKILLS.md`
- Scene/stage loading (USD) -> `skills/loading-usd/SKILLS.md`
- Step loop and frame rendering -> `skills/stepping-and-rendering/SKILLS.md`
- Reading rendered outputs -> `skills/reading-render-output/SKILLS.md`
- Writing attributes -> `skills/writing-attributes/SKILLS.md`
- Writing transforms -> `skills/writing-transforms/SKILLS.md`
- Mapping attributes/bindings -> `skills/mapping-attributes/SKILLS.md`, `skills/attribute-bindings/SKILLS.md`
- Async operations -> `skills/async-operations/SKILLS.md`
- Cloning prims -> `skills/cloning-prims/SKILLS.md`
- C project bootstrapping -> `skills/project-setup-c/SKILLS.md`
- Python project bootstrapping -> `skills/project-setup-python/SKILLS.md`
- CUDA interop -> `skills/cuda-interop/SKILLS.md`
- App-level lifecycle and ordering -> `skills/application-flow/SKILLS.md`
- Error/reporting patterns -> `skills/error-handling/SKILLS.md`

If multiple skills seem relevant, start with `skills/application-flow/SKILLS.md`, then layer in specialized skills.

## Agent Expectations

- Prefer small, targeted edits over broad refactors unless requested.
- Keep examples and skills in sync with API behavior changes.
- If you change conventions or introduce a new repeated workflow, add/update the corresponding skill under `skills/`.
- Preserve licensing headers and proprietary notices where present.
- When a `SKILLS.md` references code via `> **Source:** ...`, read the snippet markers in the referenced file to get the current code. Do not rely on stale inline examples.

### Snippet and skill rules

These rules are mandatory. Test/example code is the single source of truth; skills and docs reference it — never the other way around.

- **Adding tests or examples:** Wrap every illustrative code path in `# [snippet:name]` / `# [/snippet:name]` markers (Python) or `// [snippet:...]` (C/C++). Names are kebab-case and unique within the file. If the new code demonstrates a workflow that maps to an existing skill, add a `> **Source:**` reference in that skill. If no matching skill exists, consider creating one under `skills/`.
- **Modifying tests or examples:** Preserve existing snippet markers. If you move or restructure marked code, update the markers to stay around the illustrative section. Do not remove markers without also removing or updating every reference to them in `skills/` and `docs/`.
- **Adding skills:** Every code example in a `SKILLS.md` must come from a snippet marker in a test or example file — never write inline code blocks for API usage. If no suitable snippet exists, first add a focused test function (in `tests/test_ovrtx.py`) or example code with markers, then reference it from the skill. See `skills/README.md` for the full format.

## Notes

- The project is pre-release; behavior, APIs, and packaging details may evolve.
- `README.md` currently includes an internal-package-index note for Python installation during early access. Keep this in mind when validating install or onboarding steps.

---
> Source: [NVIDIA-Omniverse/ovrtx](https://github.com/NVIDIA-Omniverse/ovrtx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
