## blog-stan-with-splines

> The Stan models live in `code/` (e.g., `code/bsplines.stan`, `code/csplines.stan`) with companion R helpers such as `code/smoothing_diagnostics.R`. High level examples reside in `examples/`, including combined Stan/R workflows like `examples/hierarchical_regional_splines.R`. Tests are plain R scripts under `tests/`, each prefixed with `test_` and targeting a single behavior. Generated charts, CSVs, and CmdStan outputs should land in `output/`. Root-level runners (`run-code.R`, `run-examples.R`, `run-tests.R`) orchestrate minimal demos, extended analyses, and the full regression suite.

# Repository Guidelines

## Project Structure & Module Organization
The Stan models live in `code/` (e.g., `code/bsplines.stan`, `code/csplines.stan`) with companion R helpers such as `code/smoothing_diagnostics.R`. High level examples reside in `examples/`, including combined Stan/R workflows like `examples/hierarchical_regional_splines.R`. Tests are plain R scripts under `tests/`, each prefixed with `test_` and targeting a single behavior. Generated charts, CSVs, and CmdStan outputs should land in `output/`. Root-level runners (`run-code.R`, `run-examples.R`, `run-tests.R`) orchestrate minimal demos, extended analyses, and the full regression suite.

## Build, Test, and Development Commands
- `Rscript run-code.R`: Installs any missing R packages and runs the minimal spline demos, writing plots to `output/`.
- `Rscript run-examples.R`: Executes the comparative and hierarchical examples; expect longer runtimes because each Stan model compiles.
- `Rscript run-tests.R`: Sources every `tests/test_*.R` file, orders quick checks first, and reports per-script timing.
- `cmdstanr::cmdstan_model("code/bsplines.stan")`: Use within R to compile a single model when iterating on Stan code.

## Coding Style & Naming Conventions
Use two-space indentation for Stan blocks and keep identifiers in `snake_case` (`build_b_spline`, `smoothing_strength`). Comment intent with `//` before complex blocks rather than inline fragments. R scripts prefer tidyverse conventions: `<-` for assignment, `snake_case` filenames, and explicit `library()` calls in setup sections. Store reusable helpers beside their Stan counterparts in `code/`, and keep test utilities local to the test script that owns them.

## Testing Guidelines
All tests rely on `groundhog.library()` to lock package versions; add new dependencies to the vectors at the top of each test. Follow the naming format `tests/test_topic.R` and structure work as executable scripts that call CmdStan and assert diagnostics. Before opening a PR, run `Rscript run-tests.R` and confirm any new artifact paths point to `output/`. When adding Stan features, include at least one deterministic check (e.g., known function fit) to guard against regression.

## Commit & Pull Request Guidelines
Recent commits use concise, sentence-case summaries starting with a verb (`Improve hierarchical spline model ...`). Keep a single focus per commit and note material Stan or R dependency changes in the body. PRs should include: a short description of the behavior change, links to related issues, confirmation that `run-tests.R` completed, and key plot snapshots or summary tables when output shifts. Request at least one reviewer familiar with both Stan and R when modifying shared helpers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BorisVSchmid)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BorisVSchmid)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
