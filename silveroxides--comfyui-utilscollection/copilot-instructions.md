## comfyui-utilscollection

> Read `AGENTS-LOCAL.md` when present. It contains repository-machine-specific

# Agent Instructions

Read `AGENTS-LOCAL.md` when present. It contains repository-machine-specific
operating rules and must remain local only; do not stage or commit it.

Use `$comfyui-custom-node-development` for all development, testing, review, and packaging work in this repository.

If the skill is unavailable, stop and tell the user that it must be installed before continuing.

## Local ComfyUI Context

<!-- comfyui-custom-node-context:start -->
comfyui_root: C:\Users\ishim\Tools\ComfyUI
custom_nodes_root: C:\Users\ishim\Tools\ComfyUI\custom_nodes
repository_root: C:\Users\ishim\Tools\ComfyUI\custom_nodes\ComfyUI-UtilsCollection
virtual_environment_root: C:\Users\ishim\Tools\ComfyUI\.venv
python_executable: C:\Users\ishim\Tools\ComfyUI\.venv\Scripts\python.exe
<!-- comfyui-custom-node-context:end -->

## Repository-Specific Instructions

### Python module structure

- Keep `*_nodes.py` modules limited to node classes, schemas, registration
  collections, constants, and thin execution orchestration.
- Do not add module-level helper functions to `*_nodes.py`. Put algorithms,
  tensor operations, parsing, geometry, model handling, and reusable execution
  logic in a focused domain `*_helpers.py` module.
- Extend an existing helper module when its domain matches. Do not create
  catch-all helper modules or move unrelated domains into an existing helper.
- Do not extract trivial forwarding or single-expression functions unless they
  are reused or required by an external interface.
- Treat the allowlist in `tests/test_node_module_structure.py` as frozen legacy
  debt. Do not expand it. Move relevant logic out of a node module instead.
- Before adding behavior to a node module that already contains standalone
  helpers, refactor the affected helper logic into the appropriate helper
  module rather than increasing the node module's standalone-function count.

### Test selection

- Use `tests/run_tests.py` as the repository-owned test selector.
- During iteration, run an exact test or one explicit `--group`; do not run the
  complete suite after every edit.
- Before handoff, run `tests/run_tests.py --changed` once for the accumulated
  worktree changes.
- Use `tests/run_tests.py --final` only as the deliberate broader gate. Do not
  repeat it when no relevant files changed after a successful run.
- An unmapped production source file is an error. Add or correct its entry in
  `tests/test_groups.toml` instead of silently skipping coverage.
- Local untracked artifacts are intentionally excluded. Stage a new production
  source file before relying on `--changed`, or run its intended `--group`
  explicitly while creating it.

---
> Source: [silveroxides/ComfyUI-UtilsCollection](https://github.com/silveroxides/ComfyUI-UtilsCollection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
