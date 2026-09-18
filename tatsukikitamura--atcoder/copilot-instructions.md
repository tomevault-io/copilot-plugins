## atcoder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

AtCoder contest solutions in C++23 and Python. Two main areas:
- **`Algorithm/`**: Solutions for ABC/ARC/AGC contests + a C++ library
- **`Heuristic/`**: AtCoder Heuristic Contest (AHC) projects and the Fixstars speed contest

## Heuristic/Fixstars — Build & Test

This is the primary active project. It is a Fixstars speed contest: find a maximum-weight clique subject to a BOSS parameter constraint, within a 3-second time limit.

**Build:**
```bash
cd Heuristic/Fixstars
./build.sh                   # native (local dev)
./build.sh icelake           # icelake-client (contest arch, AVX-512)
./build.sh rocketlake        # rocketlake (contest arch)
```
The build script wraps CMake + make and outputs to `build/run-solver`.

**Run a single test case:**
```bash
./build/run-solver data/in-small-1.txt
# writes output to data/out-small-1.txt
```

**Run all tests (with build):**
```bash
python3 script/tests.py
python3 script/tests.py --no-build              # skip rebuild
python3 script/tests.py small                   # filter by name
python3 script/tests.py -j 4 --no-build         # parallel
```
Tests compare `data/out-{tc}.txt` against `data/golden-{tc}.txt` (if present). Without a golden file, a run is considered PASS with the score reported.

**Generate custom test cases:**
```bash
python3 script/gen.py small 42    # N=24, seed=42
python3 script/gen.py large 42    # N=64, seed=42
```

**Verify a specific output:**
```bash
python3 script/verify.py    # verifies small-1, small-2, large-1, large-2
```

## Heuristic/Fixstars — Architecture

The contest runner (`src/main.cpp`) is provided by Fixstars. It:
1. Loads input via `io_struct::LoadInput`
2. Forks a child process that calls `solve(input, output)`
3. Measures elapsed time and writes `data/out-*.txt` with `elapsed_nsec`, `PartySize`, `ParameterSum`, `PartyMembers`

**The only file to modify is `src/solve.cpp`** — it implements `void solve(input_t& input, output_t& output)`.

Data structures in `src/io_struct.hpp`:
- `input_t`: `N` (≤64), `A[MAX_N][MAX_N]` adjacency matrix, `v[MAX_N][128]` parameters, `r[128]` BOSS requirements
- `output_t`: `K_size`, `T` (ParameterSum), `members[]` (1-indexed, ascending)

`solve.cpp` uses:
- Branch-and-Bound with bitboard clique enumeration (64-bit masks, N≤64)
- SIMD (AVX-512 or AVX2 via `#include <immintrin.h>`) selected by `-march` flag
- OpenMP (`-fopenmp`) for parallel search threads
- Complement-graph greedy coloring for BnB upper bound
- `TIME_LIMIT_MS = 2850` ms budget with `std::chrono::steady_clock`

CMake flags: C++20, `-O2 -Wall -Wextra -fopenmp`, `-march=${TARGET_ARCH}` (default: `native`).

## Heuristic/AHC Projects (ahc058–ahc061 style)

Each AHC project under `Heuristic/ahcXXX/` typically contains:
- `src/main.cpp` — solver entry point
- `Makefile` — `make`, `make test`, `make fast`, `make vis`, `make gen`, `make submit`
- `scripts/test.py` — parallel test runner
- `tools/` — official Rust tester/visualizer (run with `cargo run -r`)

**Standard AHC build/test:**
```bash
cd Heuristic/ahcXXX
make                         # build
make test                    # run all test cases
make fast                    # quick test (seeds 0-9)
make vis CASE=0000.txt       # run + open visualizer
```

Compiler flags: `g++ -std=c++23 -O2 -Wall -static -DLOCAL`

## Algorithm Directory

`Algorithm/code/` and `Algorithm/Template/` contain Python and C++ snippets/templates. No build system — individual files are submitted directly to AtCoder.

## Python Environment

```bash
uv sync          # install dependencies (uses uv.lock / pyproject.toml)
```

Key Python dependencies: `optuna` (hyperparameter tuning), `numpy`, `tqdm`, `atcoder-tools`.

---
> Source: [tatsukikitamura/Atcoder](https://github.com/tatsukikitamura/Atcoder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
