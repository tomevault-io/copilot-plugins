## sapien

> If the user has not identified a concrete subsystem, read `readme.md` first, then ask which area to work on.

# AGENTS.md

## First Message
If the user has not identified a concrete subsystem, read `readme.md` first, then ask which area to work on.

When the area is clear, read the relevant docs in parallel before editing:
- Basics: `docs/source/tutorial/basic/index.md`
- Rendering: `docs/source/tutorial/rendering/index.md`
- Robotics: `docs/source/tutorial/robotics/index.md`
- Reinforcement learning: `docs/source/tutorial/rl/index.md`
- Motion planning: `docs/source/tutorial/motion_planning/index.md`
- Migration and compatibility: `docs/source/tutorial/migration/index.md`

If documentation and checked-in automation disagree, prefer the repository scripts and CI workflow over prose in `readme.md`.

## Repo Overview
- SAPIEN is a C++20 robotics simulator with optional CUDA features and Python bindings distributed as the `sapien` package.
- Current documentation lives under `docs/source`.
- `docs/source-0.1` is legacy documentation. Do not update it unless the task explicitly targets legacy docs.
- Main code areas:
  - `src/`: core C++ implementation, including `physx`, `sapien_renderer`, and shared utilities
  - `include/sapien/`: public C++ headers
  - `python/pybind/`: pybind11 bindings
  - `python/py_package/`: packaged Python API, wrappers, examples, utilities, and sensor assets
  - `pinocchio/`: Pinocchio integration
  - `cmake/`: dependency and package config helpers
  - `vulkan_shader/` and `vulkan_library/`: runtime rendering assets and bundled libraries
  - `test/`: C++ unit tests
  - `unittest/`: Python `unittest` suite
  - `manualtest/`: interactive/manual validation scripts
  - `assets/`: bundled models, robots, and data files

## Development Rules
- Keep public C++ headers, C++ implementation, Python bindings, and Python wrappers in sync when an API crosses those layers.
- When adding a new feature, add appropriate automated tests with it. Prefer Python `unittest/` coverage for Python-facing behavior and C++ `test/` coverage for native behavior.
- For Python-facing API changes, inspect all affected surfaces:
  - `include/sapien/**`
  - `src/**`
  - `python/pybind/*.cpp`
  - `python/py_package/**`
- Preserve import paths and example module paths under `python/py_package`. The documented smoke tests use `sapien.example.*`.
- Treat `manualtest/` as manual validation only. Many scripts require a GPU, Vulkan, and sometimes an onscreen display.
- Do not casually remove or rewrite large assets, shader directories, bundled libraries, or legacy docs.

## Style
- Follow `.editorconfig`:
  - `*.h`, `*.cpp`: 2 spaces
  - `*.rst`: 3 spaces
  - `CMakeLists.txt`: 4 spaces
- Follow `.clang-format` for C++ changes. The configured column limit is 99.
- Keep Python style consistent with surrounding files. This repo does not expose a stricter top-level formatter config in the checked-in docs.
- Python should be type-annotated and well-documented with comments.

## Build And Install
- Local and agent validation builds must use at most 4 compile jobs.
  - Apply the limit at invocation time with `CMAKE_BUILD_PARALLEL_LEVEL=4` or `--jobs 4`.
  - Keep repository scripts caller-configurable; do not hard-code or default the scripts themselves to 4 jobs.
- Initialize submodules before any source build:
  - `git submodule update --init --recursive`
- Preferred wheel build in Docker:
  - Use the latest `yolkarian/sapien-build-env:<tag>` image used by the checked-in CI/scripts; check `.github/workflows/build.yml` and `scripts/docker_build_wheels.sh` before building.
  - If the Docker image is not present locally, pull it from the registry first, matching GitHub Actions' container behavior:
    - `docker image inspect yolkarian/sapien-build-env:<tag> >/dev/null 2>&1 || docker pull yolkarian/sapien-build-env:<tag>`
  - Default new-feature wheel build: compile the Python 3.11 wheel with parallelism limited to 4:
    - `CMAKE_BUILD_PARALLEL_LEVEL=4 ./scripts/docker_build_wheels.sh 311`
  - General form: `CMAKE_BUILD_PARALLEL_LEVEL=4 ./scripts/docker_build_wheels.sh [39|310|311|312|313]`
- Direct build script used by CI:
  - `./scripts/build.sh [39|310|311|312|313] [--debug] [--profile] [--jobs N]`
  - When compiling directly, limit parallelism to 4 by default for validation builds:
    - `./scripts/build.sh 311 --jobs 4`
- Local install helpers exist at `scripts/install.sh` and `scripts/install_debug.sh`, but CI does not use them. Inspect them before relying on them for validation automation.
- `setup.py` drives the wheel build and invokes CMake for the native library.
- Current PhysX baseline is `107.3-physx-5.6.1`.
- CUDA support is controlled by `CUDA_PATH`. If it is unset, `setup.py` builds with `SAPIEN_CUDA=OFF`.
- PhysX 5.6.1 GPU builds require a CUDA toolkit and driver stack compatible with CUDA `>= 12.8`.
- If the matching `sapien-sim/physx-precompiled` release asset is unavailable, prefer setting `SAPIEN_PHYSX5_DIR` to a locally built PhysX tree instead of relying on FetchContent.

## Validation
- Use the narrowest validation that matches the change. Full wheel builds are expensive.
- New features must include matching automated tests unless there is a clear documented reason they cannot be tested automatically.
- After adding a new feature, the default validation path is to build the Python 3.11 wheel in the latest `sapien-build-env` image with build parallelism limited to 4, then validate that wheel inside a fresh mamba environment.
- Create a clean mamba environment for Python validation, install the newly built wheel, and run targeted tests/smoke tests there, for example:
  - `mamba create -n sapien-wheel-py311 python=3.11 -y`
  - `mamba run -n sapien-wheel-py311 python -m pip install wheelhouse/sapien-*-cp311-*.whl`
  - `mamba run -n sapien-wheel-py311 python -m unittest discover unittest`
- Python tests live under `unittest/` and use the standard library `unittest` runner.
- C++ tests live under `test/` and require configuring CMake with `-DSAPIEN_BUILD_TEST=ON`, then building the `sapien_test` target.
- Documented runtime smoke tests:
  - `python -m sapien.example.offscreen`
  - `python -m sapien.example.hello_world`
- Be careful with rendering checks on headless machines:
  - offscreen examples may emit display warnings but still succeed
  - onscreen viewer checks require a display-capable environment

## Documentation
- Build docs from `docs/`:
  - `make -C docs html`
  - `make -C docs apidoc`
- `docs/Makefile` regenerates API docs from the installed `sapien` package, so doc builds that rely on API pages assume the package imports correctly.
- If you change a public Python API, update the relevant tutorial or API-facing docs in `docs/source`.
- When examples or module names change, verify the docs still point at real files in `python/py_package/example/`.

## CI Notes
- GitHub Actions builds Linux wheels in `yolkarian/sapien-build-env:<tag>` and Windows wheels separately. Check `.github/workflows/build.yml` for the current/latest image tag instead of relying on stale hard-coded values.
- When build behavior, Python version support, or packaging details matter, check:
  - `.github/workflows/build.yml`
  - `scripts/build.sh`
  - `scripts/docker_build_wheels.sh`
  - `setup.py`
- Prefer these files over older README snippets if they disagree.

## Git Safety
- This repo may contain user changes, large binary assets, and initialized submodules. Do not use destructive cleanup commands unless explicitly asked.
- Keep commits and staging scoped to the files you actually changed.

---
> Source: [yolkarian/SAPIEN](https://github.com/yolkarian/SAPIEN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
