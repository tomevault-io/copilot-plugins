## llm-inference-optimization-lab

> This repository is a focused lab for learning and demonstrating LLM inference optimization techniques relevant to a Preferred Networks software engineering role. Keep changes small, measurable, and reproducible.

# Project Development Instructions

## Scope

This repository is a focused lab for learning and demonstrating LLM inference optimization techniques relevant to a Preferred Networks software engineering role. Keep changes small, measurable, and reproducible.

## Environment

- Work only inside this repository under `/home/chen1/projects`.
- Use Linux tools in WSL2. Do not use Windows toolchains or `/mnt/c` paths.
- Use `/usr/bin/gcc` and `/usr/bin/g++` for GCC builds.
- Configure with CMake presets and build with Ninja.
- Use the checked-in project convention of `.venv` for Python commands.
- Treat `third_party/llama.cpp` as an external, pinned Git submodule. Never edit files inside it.

## Development workflow

1. Configure native code with `cmake --preset debug` or `cmake --preset release`.
2. Build with `cmake --build --preset debug` or `cmake --build --preset release`.
3. Run native tests with `ctest --preset debug --output-on-failure`.
4. Run Python tests with `python -m pytest` from the active `.venv`.
5. Keep generated build trees, model files, and benchmark outputs untracked.

## Code quality

- Write portable C++17 and enable warnings for project-owned native targets.
- Keep native code under `src/` with public headers under `include/`.
- Put Python benchmark entry points under `benchmarks/` and tests under `tests/`.
- Do not commit fabricated benchmark data. Document hardware, software versions, model, command, and methodology with every future benchmark result.
- Add or update tests with behavioral changes.
- Keep documentation and configuration synchronized with code.

## Validation

Before handing off a change, run the relevant Debug build and both CTest and pytest suites. Also verify that the submodule is at the commit recorded in `docs/dependencies.md` and remains clean.

---
> Source: [200lz/llm-inference-optimization-lab](https://github.com/200lz/llm-inference-optimization-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
