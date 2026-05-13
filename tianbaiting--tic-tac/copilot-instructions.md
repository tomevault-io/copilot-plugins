## tic-tac

> - `src/` contains the refactored C++ implementation (`core/`, `config/`, `io/`, `interactions/`, `utils/`).

# Repository Guidelines

## Project Structure & Module Organization
- `src/` contains the refactored C++ implementation (`core/`, `config/`, `io/`, `interactions/`, `utils/`).
- `CPP/` contains the actively used solver entrypoint `CPP/run` plus legacy-compatible build assets.
- `include/` stores shared headers (`constants.h`, `type_defs.h`).
- `examples/` contains maintained end-to-end workflows (`deuteron_proton_Ay.py`, `compare_Ay_experiment.py`, `run_dpol_p_observables.py`).
- `docs/` contains method, validation, and parameter-tuning documentation.
- `data/` stores experiment inputs (`DataOfCrosssectionAndPol`) and energy/input files.
- `tests/` holds automated Python checks; `Test/` keeps older numerical harnesses.

## Build, Test, and Development Commands
- `cd CPP && make -j` builds the solver used by current workflows (`CPP/run`).
- `./CPP/run CPP/Input/input.txt` runs with an input file.
- `./CPP/run Np_WP=30 Nq_WP=30 energy_input_file=CPP/Input/tlab_190MeV.txt` runs with inline overrides.
- `python3 examples/deuteron_proton_Ay.py --work-dir output/deuteron_proton_Ay --target-tlab-mev 190` runs solver output generation for `Tlab = 190 MeV`.
- `python3 examples/compare_Ay_experiment.py --work-dir output/deuteron_proton_Ay --solver-out-dir output/deuteron_proton_Ay/solver_out --target-tlab-mev 190` validates against experiment.
- `python3 -m unittest tests/test_190mev_data_pipeline.py` runs the core regression test.

## Coding Style & Naming Conventions
- C++ target is C++17 in current build scripts.
- Follow local style in touched files; avoid repo-wide reformatting.
- Use descriptive `snake_case` names for new functions/variables in scripts and utilities.
- Keep configuration keys identical to `set_run_parameters.cpp` (`key=value`, lowercase booleans).

## Testing Guidelines
- For solver pipeline edits, run at least:
  - `python3 examples/quick_Ay_test.py`
  - `python3 -m unittest tests/test_190mev_data_pipeline.py`
- If observables scripts change, regenerate outputs under `output/` and inspect produced JSON summaries.

## Commit & Pull Request Guidelines
- Use short, imperative commit messages (`docs:`, `build:`, `examples:` prefixes are preferred).
- Keep one technical concern per commit.
- In PR descriptions, include reproducible commands and numerical impact (energy deltas, RMSE/MAE, or output file diffs).

---
> Source: [tianbaiting/Tic-tac](https://github.com/tianbaiting/Tic-tac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
