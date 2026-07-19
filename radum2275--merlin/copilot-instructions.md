## merlin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What Merlin is

Merlin is a standalone C++ solver for probabilistic inference over graphical models (Bayesian networks, Markov random fields). It supports the tasks PR (partition function / probability of evidence), MAR (posterior marginals), MAP, MMAP (marginal MAP), and EM (maximum-likelihood parameter learning for Bayes nets only). Inference algorithms include Loopy Belief Propagation (LBP), Iterative Join-Graph Propagation (IJGP), Join-Graph Linear Programming (JGLP), Weighted Mini-Bucket (WMB), Bucket-Tree Elimination (BTE), Clique-Tree Elimination (CTE), and Gibbs sampling. WMB/IJGP/JGLP are parameterized by an **i-bound** that trades accuracy for cost (i-bound ≥ treewidth gives exact inference).

Requires **boost** (linked against `boost_program_options`, and `boost_thread` via CMake). Built with CMake at C++17.

## Build & run

**CLI binary (CMake — the only build for the solver):**
```
mkdir build && cd build
cmake ..
cmake --build .        # produces build/merlin
```

**Python bindings** (`src/merlin_pybind.cpp`, opt-in CMake target): configure with `-DMERLIN_BUILD_PYTHON=ON`; pybind11 is fetched via `FetchContent` (see `python/CMakeLists.txt`), so no preinstalled env is needed. Builds `build/python/merlin.*.so`, importable as `import merlin` (add `build/python` to `PYTHONPATH`). The module exposes the `Merlin` class plus `Task`/`Algorithm`/`InputFormat`/`OutputFormat` IntEnums. `run()` writes the solution to `<output_file>.<TASK>` (there is no in-memory output string), so the Python workflow reads that file back — see the README example.

**Docs** (Doxygen, opt-in CMake target): configure with `-DMERLIN_BUILD_DOCS=ON` then `cmake --build build --target docs` (requires Doxygen installed). `merlin.doxyfile` is the config; it scans `include`, `src`, and `README.md` (the mainpage) and writes HTML to `docs/html/`. Generated output under `docs/` is gitignored. Can also be run directly with `doxygen merlin.doxyfile`.

**Tests** (GoogleTest, in `tests/`). The root CMake build enables them by default (`MERLIN_BUILD_TESTS=ON`) and fetches GoogleTest via `FetchContent` on first configure (needs network once). To build and run:
```
cmake -S . -B build && cmake --build build -j
ctest --test-dir build --output-on-failure          # all tests (~0.4s)
./build/tests/merlin_unit_tests --gtest_filter=Factor.*   # one suite directly
```
`tests/unit/` covers the header value classes (`variable`, `variable_set`, `factor`, `graph`, indexing helpers, `my_set`, etc.) and `graphical_model` UAI parsing; `tests/integration/` runs `Merlin::run()` end-to-end and checks the MAR/PR result against the committed `cancer.uai.MAR` golden and EM against a pinned snapshot in `tests/fixtures/`. The test targets recompile the solver `src/*.cpp` (minus `main.cpp`/`merlin_pybind.cpp`) directly. Configure with `-DMERLIN_BUILD_TESTS=OFF` for a plain binary build with no GoogleTest dependency. There is no linter.

Two latent library quirks are documented (not worked around) by the tests: the `MapType&` overload of `ind2sub` (`variable_set.h`) is declared `void` but `return`s a value (rejected by strict compilers), and `MER_ENUM`'s string constructor only prefix-matches the first value and values right after a comma — later values are stored with a leading space (see `tests/unit/test_enum.cpp`).

You can also verify changes by running the binary against instances in `examples/`, e.g.:
```
./build/merlin --input-file examples/pedigree1.uai --evidence-file examples/pedigree1.evid \
         --task MAR --algorithm wmb --ibound 4 --iterations 10 --output-format json
```
Output is written to `<input>.<TASK>.out` (or a `--output-file`); the EM task writes a new model to `<input>.EM`. See README.md for the full CLI flag list and the UAI file formats (input/evidence/virtual-evidence/query/dataset/output) — these formats are the authoritative interface contract and the README documents them in detail.

## Architecture

**Two entry points, one engine.** The CLI (`src/main.cpp` → `src/program_options.cpp`) and the Python module (`src/merlin_pybind.cpp`) both drive the same `Merlin` facade class (`include/merlin.h`, `src/merlin.cpp`). `main.cpp` parses options into a struct and calls a long sequence of `eng.set_*(...)` setters followed by `eng.run()`; the pybind module exposes those same setters/getters as Python properties. **When you add a configurable parameter, it must be threaded through all of: the `Merlin` member + setter/getter, `program_options` (CLI), and `merlin_pybind.cpp` (Python), to stay consistent across both front ends.**

**Dual I/O modes.** `Merlin` reads models/evidence/query/dataset either from files or from in-memory strings, selected by `set_use_files(bool)`. Every input has both a `*_file` and a `*_string` variant (e.g. `read_model(const char*)` vs `read_model(std::string)`). The string path exists primarily for the Python/embedded use case.

**Algorithm class hierarchy.** `include/algorithm.h` defines the pure-virtual `algorithm` interface (in `namespace merlin`): `init()`, `run()`, `logZ()/logZub()/logZlb()`, `ub()/lb()`, `best_config()`, and `belief(...)`. Each concrete algorithm is a class that inherits **both** `graphical_model` and `algorithm` (e.g. `class wmb: public graphical_model, public algorithm`), paired as `include/<algo>.h` + `src/<algo>.cpp` for `wmb`, `ijgp`, `jglp`, `lbp`, `bte`, `cte`, `gibbs`. `Merlin::run()` dispatches on the selected task+algorithm, constructs the right algorithm object, feeds it the model and evidence, runs it, and formats the solution (UAI or JSON). EM lives in `em.h/em.cpp` and internally uses CTE for exact inference.

**Model representation.** `graphical_model` (`include/graphical_model.h`) is the base data structure — a collection of `factor`s over `variable`/`variable_set`s — and extends `graph` (`include/graph.h`). Supporting value-type headers: `factor.h`, `factor_graph.h`, `variable.h`, `variable_set.h`, `index.h`, `indexed_heap.h`, `set.h`, `vector.h`, `observation.h` (used for EM training examples). Most of these are header-only.

**Constants & enums.** All task/algorithm/format codes are `#define`s in `include/base.h` (`MERLIN_TASK_*` = 10/20/30/40/50, `MERLIN_ALGO_*` = 1000–1009, `MERLIN_INPUT_*`, `MERLIN_OUTPUT_UAI/JSON`). The Python `Task`/`Algorithm`/`InputFormat`/`OutputFormat` IntEnums in `merlin_pybind.cpp` are built by hand from these same values — **keep them in sync when editing `base.h`.** `include/enum.h` is a separate string-izable enum macro (`MER_ENUM`) used internally for parsing/printing, unrelated to the `base.h` codes.

## Conventions

- Library code lives in `namespace merlin`; the top-level `Merlin` facade and the `MERLIN_*` macros are global.
- Header/source pairs: interface in `include/`, implementation in `src/`; several utility classes are header-only.
- New solver source files must be added to the shared `MERLIN_SOLVER_SRCS` list in `CMakeLists.txt` — one place feeds the `merlin` executable, the test targets (`tests/CMakeLists.txt`), and the Python module (`python/CMakeLists.txt`). `src/main.cpp` (CLI entry point) and `src/merlin_pybind.cpp` (Python module) are intentionally kept out of that list and added only to their respective targets.
- `AOBB`, `AOBF`, `RBFAOO` algorithm codes exist but are **not implemented**.

---
> Source: [radum2275/merlin](https://github.com/radum2275/merlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
