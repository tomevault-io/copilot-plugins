## trafficassignmentapproaches

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build System

C++20 project using CMake + Ninja on Linux. Dependencies are installed via apt: Eigen3, Boost.Multiprecision, NLopt. toml++ and OptimLib are fetched at configure time via CMake `FetchContent`.

```bash
# Prerequisites (Ubuntu/Debian)
sudo apt install build-essential cmake ninja-build pkg-config \
                 libeigen3-dev libnlopt-cxx-dev libboost-all-dev

# Configure
cmake --preset linux-release    # or linux-debug / linux-relwithdebinfo

# Build
cmake --build --preset build-release  # or build-debug / build-relwithdebinfo

# Run
./build/linux-release/cndp_solver --config configs/cnd.siouxfalls.toml
./build/linux-release/cndp_solver --help
```

Two executables: `cndp_solver` (CNDP solver) and `tap_solver` (standalone TAP solver). Build output goes to `build/<preset-name>/`. No automated tests — validation is done by running experiments and inspecting metrics outputs.

## Architecture

### Domain Layers

```
cndp_solver.cpp                   — Config parsing, wiring, execution
    ↓
cnd/BilevelCND                    — Upper-level optimization over link capacities
    ↓
cnd/OptimizationPipeline          — Sequential [[pipeline]] execution
    ├── NloptOptimizationStep     — NLopt algorithms (COBYLA, BOBYQA, ISRES, ...)
    ├── OptimalityConditionStep   — Optimality condition-based iteration
    └── OptimlibOptimizationStep  — Population-based metaheuristics (DE, PSO, ...)
    ↓
cnd/CndOptimizationContext        — Objective evaluation (TotalTravelTime + BudgetCost)
    ↓
tap/algorithms/                   — Lower-level TAP solvers (user equilibrium)
    ↓
tap/core/Network                  — Network topology, Dijkstra, OD pairs
tap/data/Link                     — BPR delay function: t(x) = t0*(1 + b*(x/c)^p)
```

### Traffic Assignment Approaches (`include/tap/algorithms/`)

Two solver implementations, both implementing `TrafficAssignmentApproach<T>`:

1. **RouteBasedApproach** — Explicit path enumeration; factory-based shift methods (`NewtonStep`, `Krylatov2023`). Supports parallel route search via `route_search_threads`.

2. **TapasApproach** — Bush-based TAPAS algorithm (TAsK-style); fastest, reaches machine precision (~1e-15 RGAP). Uses Halley step with Armijo backtracking, stall detection with bush restart, random origin shuffle.

### Bilevel CNDP (`include/cnd/`)

- **BilevelCND** — Pipeline-based constructor only: takes `std::vector<OptimizationStepConfig>`. Iterates optimization steps ↔ TAP solver to design link capacities minimizing system travel cost under budget constraints.
- **CndOptimizationContext** — Shared evaluation context with crash recovery (SIGSEGV handler). Includes `GetAlgorithmName()` and `SupportsHardBudgetConstraint()`.
- **OptimizationStepFactory** — Creates steps from config: `"nlopt"`, `"optimality_condition"`, `"optimlib"`.
- **CndStatisticsRecorder** — Records trace CSV (quality vs time), metadata JSON, summary CSV (append-only). Summary CSV has a `Scenario` column.
- **DirectedConstraintLoader** — Reads per-link capacity constraints from CSV.

## Configuration

TOML-based layered config system with precedence: **defaults → TOML file → environment variables → CLI args**.

```bash
./build/linux-release/cndp_solver \
  --config ./configs/cnd.siouxfalls.toml \
  --route-threads 2 \
  --max-standard-iters 150
```

Environment variable prefix: `CND_` (e.g., `CND_ROUTE_THREADS=1`). CLI metrics args prefixed with `metrics_` (e.g., `--metrics_scenario_name`).

### TOML Sections

**`[network]`** — `dataset`, `constraints_file`.

**`[solver]`** — Algorithm selection: `approach` (RouteBased/Tapas), `approach_alpha`, `max_standard_iterations`, `budget_function_multiplier`, `budget_upper_bound`, `link_capacity_selection_threshold`, `budget_threshold`.

**`[solver.tapas]`** — Tapas-specific: `mu`, `v`, `pas_per_origin`, `pas_multiplier`, `rgap_check_interval`.

**`[solver.route_based]`** — RouteBased-specific: `shift_method`, `route_search_threads`, `full_iteration_count`, `origin_iteration_count`, `ema_alpha`.

**`[output]`** — `verbose`, `progress_format`, `print_effective_config`, `quiet`.

**`[metrics]`** — Output control: `enable_trace`, `output_root`, `write_metadata_json`, `write_summary_csv`, `scenario_name`, `append_dataset_subdir`.

**`[[pipeline]]`** — Array of optimization steps executed sequentially:
- `type = "nlopt"` — `algorithm` (LN_COBYLA, LN_BOBYQA, GN_ISRES, etc.), `max_iterations`, `tolerance`, optional `local_algorithm`
- `type = "optimality_condition"` — `max_iterations`
- `type = "optimlib"` — `algorithm` (DE, DE_PRMM, PSO, PSO_DV, NM, GD), `max_iterations`, `population_size` (0 = auto)

At least one `[[pipeline]]` entry is required; `cndp_solver.cpp` errors if none found.

### Structured Config Types (C++)

Config is loaded via `include/common/TomlConfigLoader.h` into structured types:
- `Config::CndpConfig` — used by `cndp_solver.cpp` (has `NetworkConfig`, `SolverConfig`, `OutputConfig`, `CndMetricsConfig`, `vector<OptimizationStepConfig>`)
- `Config::TapConfig` — used by `tap_solver.cpp` (subset: `NetworkConfig`, `SolverConfig`, `OutputConfig`)

### Comparison Manifests

Comparison manifests in `configs/comparisons/*.toml` use inline scenarios with a `[defaults]` + `[[scenarios]]` pattern. Each scenario defines its `[[scenarios.pipeline]]` and optional per-scenario overrides. The runner (`run_cndp_comparison.py`) deep-merges defaults with scenario overrides and generates per-scenario TOML files at runtime.

## Key File Locations

- `src/cndp_solver.cpp` — CNDP entry point; all config/wiring logic
- `src/tap_solver.cpp` — Standalone TAP solver entry point
- `include/common/TomlConfigLoader.h` — TOML config loader + structured config types + env/CLI overrides
- `include/common/ConfigUtils.h` — CLI parsing, path resolution, string utilities
- `include/cnd/BilevelCND.h` — Bilevel optimization core
- `include/cnd/CndOptimizationContext.h` — Shared context, objective evaluation, crash recovery
- `include/cnd/OptimizationPipeline.h` — Sequential step execution
- `include/cnd/steps/` — Step implementations (Nlopt, OptimalityCondition, OptimLib)
- `include/tap/algorithms/common/TrafficAssignmentApproach.h` — Base TAP interface
- `include/tap/core/Network.h` — Network data structure + Dijkstra
- `include/tap/data/Link.h` — BPR function implementation
- `configs/cnd.siouxfalls.toml` — Reference CNDP config
- `configs/comparisons/` — Comparison manifests with inline scenarios

## Scripts

- `scripts/run_experiment.py` — Unified experiment runner for TAP and CNDP. Creates self-contained run folders with CSV outputs and publication-ready plots. Subcommands: `tap` (standalone TAP), `cndp` (single CNDP scenario).
- `scripts/run_cndp_comparison.py` — Batch runner: reads TOML comparison manifests with inline scenarios, generates per-scenario TOML configs at runtime, launches experiments, organizes outputs into self-contained run folders under `performance_results/{Dataset}/runs/`.
- `scripts/plot_cndp_comparison.py` — Publication-ready analysis: objective vs time plots, convergence, sensitivity, summary tables (CSV + LaTeX). Supports `--run-dir` mode and multi-run averaging.

## Data Format (TNTP)

Network files (`*_net.csv`): `init_node, term_node, capacity, length, free_flow_time, b, power, speed, toll, link_type`. Trip matrices (`*_trips.csv`): OD demand matrix. Datasets live in `data/TransportationNetworks/` (SiouxFalls, Anaheim, Barcelona).

## Code Conventions

- **Headers-only design:** Most implementation is in `.h` files under `include/`.
- **Templates on numeric type:** `template<typename T>` parameterized (typically `double`, `long double`, or Boost high-precision).
- **Factory pattern:** Used for algorithm selection — `RouteBasedShiftMethodFactory`, `OptimizationStepFactory`.
- **Pipeline-based optimization:** All CNDP configs must use `[[pipeline]]` entries; the legacy single-algorithm constructor was removed.

---
> Source: [baspresso/TrafficAssignmentApproaches](https://github.com/baspresso/TrafficAssignmentApproaches) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
