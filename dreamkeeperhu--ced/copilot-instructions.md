## ced

> This repository contains a C++17 metaheuristic optimization solver for Cloud-Edge-Device (CED) task scheduling. It builds a single executable (`CED_Schedule`) and supports multiple solvers (GA, DE, GDE, CCHIHH). The repo also includes Python scripts for running experiment batches and plotting results, plus sample datasets and precomputed outputs.

# AGENTS.md

## Overview
This repository contains a C++17 metaheuristic optimization solver for Cloud-Edge-Device (CED) task scheduling. It builds a single executable (`CED_Schedule`) and supports multiple solvers (GA, DE, GDE, CCHIHH). The repo also includes Python scripts for running experiment batches and plotting results, plus sample datasets and precomputed outputs.

## Tech Stack
- Language: C++17
- Build: CMake (3.15+), Visual Studio 2019/2022 on Windows
- Scripts: Python (for experiment runs and plotting)
- Outputs: CSV/PNG/PDF result artifacts

## Project Structure
- `src/` C++ implementation (.cpp)
- `include/` C++ headers (.h)
- `data/` Input datasets (txt)
- `scripts/` Experiment runners and plotting utilities (Python)
- `results/` Generated outputs (csv/txt/png/pdf)
- `build/` CMake build output (local)
- `Final/` Final figures (png)
- `CMakeLists.txt` Build configuration
- `README.md` Build and usage guide

## Development Setup
- Configure and build (Windows):
  - `cmake -S . -B build -G "Visual Studio 17 2022" -A x64`
  - `cmake --build build --config Release`
- The binary is `build/Release/CED_Schedule.exe`.
- Data files are copied into `build/data` during CMake configure.

## Running the Solver
- Minimal run:
  - `./build/Release/CED_Schedule.exe --data_dir ./data --data_file data_matrix_100.txt --seed 1`
- Solver selection via `--solver` (GA, DE, GDE, CCHIHH).
- Use `--help` to view all CLI options.

## Coding Standards and Conventions
- Keep C++ source in `src/` and headers in `include/`.
- If you add or rename files, update `CMakeLists.txt` lists.
- Prefer deterministic runs via `--seed` for reproducibility.
- Use existing argument parsing patterns in `src/main.cpp` when adding new CLI options.

## Testing and Validation
- There is no formal test suite.
- Use the small sample dataset (`data/data_matrix_100.txt`) for quick sanity checks.
- For algorithm changes, run a short generation count and compare best fitness across seeds.
- Python plotting scripts in `scripts/` can visualize convergence and variance.

## Data and Results
- Input datasets live in `data/`.
- `results/` contains many generated artifacts; avoid modifying unless re-running experiments.
- If you add new experiments, document the command used and output location.

## Decision Trees

### I need to add a new solver
1. Implement logic in a new `.cpp` and `.h` under `src/`/`include/`.
2. Wire it into `src/main.cpp` argument handling.
3. Add files to `CMakeLists.txt`.
4. Run a short sanity test with `--generations 5` and a fixed seed.

### I need to run experiments or create plots
1. Check `scripts/` for an existing runner or plotter.
2. Ensure Python dependencies (matplotlib/pandas/numpy) are available.
3. Run scripts from repo root so relative paths resolve.
4. Save outputs into `results/` and note the run parameters.

### I need to change dataset format
1. Update parsing logic in `src/Problems.cpp`.
2. Validate on a small input file in `data/`.
3. Update any scripts that assume the old format.

## Deployment
- This is a local research executable; there is no deployment pipeline.
- Release builds are produced via CMake and distributed as the executable plus data files.

---
> Source: [dreamkeeperhu/CED](https://github.com/dreamkeeperhu/CED) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
