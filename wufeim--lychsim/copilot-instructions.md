## lychsim

> This file is the contributor and AI-agent guide for the LychSim repository. **Both human contributors and AI assistants** (Claude, Claude Code, and similar agents) working on this codebase or its forks should treat the conventions below as authoritative -- they keep the public API, documentation, and tests aligned across changes.

# CLAUDE.md

This file is the contributor and AI-agent guide for the LychSim repository. **Both human contributors and AI assistants** (Claude, Claude Code, and similar agents) working on this codebase or its forks should treat the conventions below as authoritative -- they keep the public API, documentation, and tests aligned across changes.

For the project pitch and example usage, see [README.md](README.md). For the v1.0.0 feature surface and planned future work, see [ROADMAP.md](ROADMAP.md).

## Quick Start for Contributors

```bash
git clone https://github.com/wufeim/LychSim.git
cd LychSim
pip install -e .[docs]              # package + docs extras (Sphinx, theme, sphinx-design)
```

Most code paths assume a running LychSim / UnrealCV instance. Quickest way: download a pre-built scene binary and launch it via [`lychsim run`](https://wufeim.github.io/LychSim/docs/cli.html) -- see the [Quick Start tutorial](https://wufeim.github.io/LychSim/tutorials/quick_start.html). For UE5 setup from source, see the [installation guide](https://wufeim.github.io/LychSim/tutorials/installation.html).

```bash
pytest tests/                                 # unit + integration tests (need a running UE)
black src/ && flake8 src/                     # format + lint
cd docs && make html                          # build the docs to docs/_build/html
```

## Architecture

LychSim is layered:

- **Communication layer** -- [`api/client.py`](src/lychsim/api/client.py): low-level socket client implementing the UnrealCV binary protocol. Thread-safe request/response.
- **Mid-level API** -- [`api/api.py`](src/lychsim/api/api.py): `UnrealCv_API` maps Python methods to raw text commands.
- **High-level wrapper** -- [`api/wrapper/`](src/lychsim/api/wrapper/): `LychSim`, the user-facing class, composed from three mixins:
  - [`CameraCommandsMixin`](src/lychsim/api/wrapper/camera_mixin.py) -- image capture (RGB / depth / normal / segmentation / point map / z-buffer), pose, intrinsics.
  - [`ObjectCommandsMixin`](src/lychsim/api/wrapper/object_mixin.py) -- list / spawn / delete / query / update actors, bounding boxes, per-object annotations.
  - [`DataCommandsMixin`](src/lychsim/api/wrapper/data_mixin.py) -- pause / resume, debug-line drawing.
- **Core data structures** -- [`core/`](src/lychsim/core/): `Object`, `AABB`, `OBB`, `SemanticScene` / `SemanticLevel` / `SemanticRegion`. Serializable to dict and `.npz`.
- **CLI** -- [`cli/`](src/lychsim/cli/): the `lychsim` command. See the [CLI reference](docs/docs/cli.rst) for every subcommand.
- **MCP server** -- [`mcp/`](src/lychsim/mcp/): a thin FastMCP layer exposing the wrapper to LLM clients (Claude Code, Claude Desktop) over stdio. One tool module per mixin under [`mcp/tools/`](src/lychsim/mcp/tools/).
- **Scripts** -- [`scripts/`](scripts/): standalone utilities that connect to a running LychSim instance.

The high-level wrapper is the canonical API. Contributor code should add methods to the appropriate mixin rather than calling `client.request(...)` directly.

## Coordinate System & Geometry Conventions

Unreal Engine (and therefore LychSim) uses a **left-handed, Z-up** coordinate system. All positions are in **centimeters**, rotations in **degrees** as `[pitch, yaw, roll]`.

### Bounding box flavors

The C++ plugin exposes four bbox shapes per object via `get_obj_aabb` / `get_obj_obb` / `get_obj_annots`:

- **`aabb`** -- `FActorController::GetAxisAlignedBoundingBox()`. Typically reflects the root / collision component only and understates Blueprint composites.
- **`bounds`** -- `Actor->GetActorBounds(false, ...)`. Aggregates *all* registered primitive components, including non-colliding visual meshes. **Use this for visual alignment** (spawn offsets, stacking).
- **`bounds_tight`** -- `Actor->GetActorBounds(true, ...)`. Only colliding components.
- **`obb`** (Python `get_obj_obb`) -- same data as `bounds`, returned as flat `center extent rotation`.

When in doubt, prefer `bounds` / `get_obj_obb` over `aabb`.

### Spawn offset convention

Assets have their pivots at arbitrary positions. To place an object so its visual bbox bottom-center lands on a desired world location:

1. Pre-compute the offset by spawning the asset at origin with rotation 0, querying `bounds`: `offset = (-cx, -cy, ez - cz)`.
2. At spawn time: `location = desired_loc + offset`, `yaw = desired_yaw + annotated_calibration_rotation`.

The pre-computed `mesh_offset_{x,y,z}` and `mesh_extent_{x,y,z}` columns ship with the [`wufeim/lychsim_objects`](https://huggingface.co/datasets/wufeim/lychsim_objects) Hugging Face dataset.

## Code Conventions

- **Python ≥3.10** required -- the wrapper uses PEP 604 union syntax (`X | Y`) at function-definition time without `from __future__ import annotations`.
- **`numpy==1.26.4` is pinned**. Don't change this version without a coordinated upgrade of the rest of the package.
- **Public surface lives on `LychSim` mixins.** New methods go into the appropriate mixin; raw `client.request(...)` calls stay inside the wrapper.
- **Docstrings are Google-style** with `Args:` / `Returns:` / `Raises:` blocks. Sphinx is configured with `sphinx.ext.autodoc` + `sphinx.ext.napoleon` ([docs/conf.py](docs/conf.py)) so these blocks render as proper field lists.
- **Daemon thread invariant.** [`api/client.py`](src/lychsim/api/client.py) creates its receive thread with `daemon=True` so forgetting to call `disconnect()` does not hang the interpreter. Explicit `disconnect()` still shuts the thread down via the `recv_num_q` sentinel -- do not remove the daemon flag.
- **Commit style.** Use the existing prefix convention -- `feat(scope):` / `fix(scope):` / `docs(scope):` / `refactor(scope):` / `chore(scope):`. `git log --oneline` shows the established patterns.

## Documentation Conventions

When adding or modifying public API surface, keep the documentation in lockstep -- the docs site is built from autodoc and breaks silently when surface drifts.

- **Docstrings on new public methods, classes, and module-level functions.** Google-style with `Args:` / `Returns:` / `Raises:` blocks; Sphinx runs `sphinx.ext.autodoc` + `sphinx.ext.napoleon`, and those blocks become the rendered field lists. Call out units (cm, degrees) and any non-obvious return-shape conventions (e.g. the `{"status", "outputs": [...]}` envelope). Cross-link siblings with `:meth:` / `:class:` so the page is navigable.
- **New `lychsim` CLI subcommands** need an entry in [docs/docs/cli.rst](docs/docs/cli.rst), placed under the right workflow group (Lifecycle / Data capture / Registries / Diagnostics) with at least one example invocation. The bullet list at the top of that page also needs the new `:ref:` entry.
- **New top-level public modules, classes, or mixins** need an entry in [docs/docs/python_api.rst](docs/docs/python_api.rst). The page is structured as **flat H2 sections** -- one per logical group -- so every section surfaces in the pydata-sphinx-theme sidebar. Promote new groups to H2 rather than nesting under H3.
- **RST gotcha:** section underline characters (`=` / `-` / `~`) must be **at least as long as the title text** or docutils errors. CI runs Sphinx with `-W` (warnings as errors), so this fails the deploy.
- **CI mechanics.** The deploy workflow ([.github/workflows/deploy-docs.yml](.github/workflows/deploy-docs.yml)) runs `pip install -e .[docs]` on Python 3.11. New runtime imports in the wrapper must be reachable from the package's main `dependencies` (autodoc imports them at build time); new Sphinx extensions go in the `[docs]` optional-dependency in [pyproject.toml](pyproject.toml).
- **Local sanity check.** `cd docs && make html` and skim `_build/html/docs/python_api.html` before pushing.

## How to Add ____

### A new wrapper method

1. Pick the right mixin: `camera_mixin.py`, `object_mixin.py`, or `data_mixin.py`.
2. Implement the method; call `self.client.request("lych ...")`. Return native Python types where possible.
3. Write a Google-style docstring (Args / Returns / Raises). Mention units, cross-link siblings.
4. If the method introduces a new public concept, add or extend a section in [docs/docs/python_api.rst](docs/docs/python_api.rst).

### A new CLI subcommand

1. Add a module under [src/lychsim/cli/commands/](src/lychsim/cli/commands/) following the pattern of an existing one (e.g. [`capture.py`](src/lychsim/cli/commands/capture.py)). Required exports: `HELP` (string), `add_arguments(parser)`, `run(args)`.
2. Register it in [src/lychsim/cli/main.py](src/lychsim/cli/main.py), placed in the workflow ordering (lifecycle → data → registries → diagnostics).
3. Document it in [docs/docs/cli.rst](docs/docs/cli.rst) under the right workflow section, plus the `:ref:` bullet list at the top.

### A new MCP tool

1. Pick or create the right module under [src/lychsim/mcp/tools/](src/lychsim/mcp/tools/) (one per mixin).
2. Decorate the function with `@mcp.tool()`. Import `mcp` and `get_sim` from `..server`. Call `get_sim()` lazily -- never instantiate `LychSim` at module import (Claude Code spawns the MCP server before UE is up).
3. Add an import for the new tool module at the bottom of [src/lychsim/mcp/server.py](src/lychsim/mcp/server.py) so the decorator fires on startup.
4. Write a thorough docstring -- it becomes the tool description the LLM sees. Include units and coordinate conventions; reference sibling tools by name.

### A new test

Tests live under [tests/](tests/) and require a running LychSim / UnrealCV server. Default endpoint is `localhost:9000`; override per-run with `pytest tests/ --server_name <ip> --port <port>`. Temporary outputs go to `tests/tmp/` (gitignored).

## Pull Request Workflow

1. **Open an issue first** for any user-visible change -- feature request, bug, or breaking change. This lets us discuss scope before code is written.
2. **Branch off `main`.** Match the existing commit-message style.
3. **Run tests locally** if you can (requires a running UE instance). Run `black src/ && flake8 src/` before pushing.
4. **Update the documentation** alongside code changes -- see *Documentation Conventions* above.
5. **Update [ROADMAP.md](ROADMAP.md)** if your change ships a user-visible feature for the next release.
6. **Open a PR against `main`** with a description that links the issue.

Contributions are accepted under the project's **MIT License**. By submitting a PR you agree your contribution is licensed under MIT.

## Where to Look Next

- [README.md](README.md) -- project pitch, example usage, dataset pointers.
- [ROADMAP.md](ROADMAP.md) -- v1.0.0 feature surface and planned future work.
- [Quick Start tutorial](https://wufeim.github.io/LychSim/tutorials/quick_start.html) -- get a running scene without a UE5 install.
- [CLI reference](docs/docs/cli.rst) -- every `lychsim` subcommand and its flags.
- [Python API reference](docs/docs/python_api.rst) -- auto-generated from the docstrings on the wrapper, core types, and utility helpers.

---
> Source: [wufeim/LychSim](https://github.com/wufeim/LychSim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
