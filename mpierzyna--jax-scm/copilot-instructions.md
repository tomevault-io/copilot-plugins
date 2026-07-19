## jax-scm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

JAX-SCM is a Single Column Model (SCM) for atmospheric boundary layer simulation, implemented in JAX for differentiability and JIT compilation. It implements the MYNN 2.5 turbulence closure scheme with Monin-Obukhov surface layer coupling. The code is designed for research use with ERA5/CERRA reanalysis data integration.

## Commands

This project uses `uv` for package management. Always prefix Python/tool invocations with `uv run`.

```bash
# Install (editable)
uv sync

# Run tests
uv run pytest tests/

# Run a single test file
uv run pytest tests/test_e2e.py

# Run a single test
uv run pytest tests/test_e2e.py::test_name

# Launch interactive UI
uv run scm-ui

# Run a script
uv run python some_script.py
```

## Architecture

### Key Abstractions (`src/scm/interfaces.py`)

Concrete types (`ProgVarsMYNN`, `DiagVarsMYNN`, etc.) are defined in `scm.mynn.interfaces` and re-exported from `scm.interfaces`; swapping closure schemes means updating only those imports.

- **`Simulation`** — plain dataclass (not JAX-registered): grid, mo_settings, init (`ProgVarsMYNN`), forcing, th_ref, time bounds, optional `t_index`
- **`Forcing`** — frozen dataclass: u_geo, v_geo (`ForceSingleFn`), f_c, w_th_s or th_s (mutually exclusive), w_qv_s, dth_dz_top, ls_tends (`ForceTendsFn | None`)
- **`Output`** — JAX-registered dataclass: state_traj (`ProgVarsMYNN`), diag_traj (`DiagVarsMYNN`), mo_traj (`MOResult`), t_s
- **`ModelFn`** protocol — `(t_s, state, params) → (tendencies, diag, mo_result)`, the ODE right-hand side
- **`ClosureFn`** protocol — `(state, grads, mo_res, params) → DiagVarsMYNN`; params explicit for `jax.grad`
- **`ForceSingleFn`** protocol — `(t_s) → scalar or (Nz,) array`; used for all time-varying forcing fields
- **`ForceTendsFn`** protocol — `(t_s, state, grads, diag) → ProgVarsMYNN`; large-scale tendencies

### Physics Modules

- **`src/scm/mynn/`** — MYNN 2.5 closure. `ProgVarsMYNN` holds u, v, th, qv, qke (note: qke = q² = 2×TKE). `closure.py` derives length scales and eddy diffusivities; references Nakanishi & Niino 2009 (NN09). `model.py` assembles the full model function via `init_model()`. `io.py` provides `sim_from_ds()` to reconstruct a `Simulation` from a saved xarray Dataset.
- **`src/scm/mo.py`** — Monin-Obukhov surface layer. `init_mo_sfc()` builds an iterative solver. `MOResult` contains friction velocity, surface fluxes, Obukhov length, and stability parameter. `BusingerDyerSimFuncs` provides configurable stability functions (phi_m, phi_h, psi_m, psi_h); `BusingerDyerAltSimFuncs` subclass allows independent momentum/heat parameters.
- **`src/scm/grid.py`** — `StaggeredGrid` with full levels (cell centers) and half levels (cell faces). Full levels: `z = dz * (0.5, 1.5, ...)`, half levels: `zh = dz * (0, 1, ...)`.
- **`src/scm/grad.py`** — `d_dz()`: 1st-order finite differences from full to half levels.
- **`src/scm/consts.py`** — Physical constants: g=9.81, kappa=0.4, rho_0=1.225, cp=1005.0, Rd=287.0, L_v=2257e3, qke_min=1e-10.
- **`src/scm/convert.py`** — Thermodynamic utilities: virtual temperature (tv_to_t, t_to_tv), flux conversions (w_th_to_w_thv, w_thv_to_w_th), pressure/density profiles (p_rho_from_th, p_rho_from_tk), geostrophic wind from geopotential (uv_geo_from_z), Coriolis parameter (get_fc).

### Time Integration (`src/scm/time_stepping/`)

- **`base.py`** — `simulate()`: unified entry point using `jax.lax.scan` for JIT-compatible looping
- **`explicit.py`** — Euler (warmup) and Adams-Bashforth 2 (AB2); adaptive AB2 with CFL-based timestep `dt = CFL_max * dz² / K_max`
- **`implicit.py`** — Crank-Nicolson semi-implicit scheme; solves a tridiagonal system per variable using `jax.lax.linalg.tridiagonal_solve`
- **`utils.py`** — state clipping to physical bounds (e.g., qke ≥ qke_min, qv ≥ 0); critical for stability

### Data Flow

```
Simulation + Forcing → init_model() → ModelFn
                                          ↓
                       simulate() [lax.scan over timesteps]
                         1. ModelFn(t_s, state) → tendencies
                         2. Time stepper (AB2 or CN) → new state
                         3. Clip to physical bounds
                          ↓
                       Output → out_to_ds() → xarray Dataset → NetCDF
```

### Supporting Modules

- **`src/scm/config/`** — Pydantic namelist model (`Namelist`); `load_namelist(yaml)` for YAML config. `TimeIntMethod` StrEnum selects IMPLICIT or EXPLICIT. `AdaptiveTimestepConfig` holds CFL_max and dt_s_max; ignored for implicit schemes.
- **`src/scm/forcing/`** — ERA5 (`era5.py`) and CERRA (`cerra/`) reanalysis interfaces; `interp.py` for temporal/vertical interpolation; `utils.py` for `sample_forcing`.
- **`src/scm/io/`** — `local.py`: `out_to_ds()` converts `Output` to xarray Dataset; `cache.py` for ERA5 data caching.
- **`src/scm/examples/gabls1.py`** — `get_gabls1()` builds the GABLS1 reference Simulation (Cuxart et al. 2006): stably stratified, Nz=64, H=400 m, 9-hour run with surface cooling.
- **`src/scm/reporter/`** — HTML diagnostic reports
- **`src/scm_ui/`** — Bokeh web app and Click CLI
- **`validation/`** — Reference cases against literature: GABLS1, GABLS3, GABLS3-ERA5, Andren 1994, Wangara day 33. Each has a `run.py`, output NetCDF, and HTML report.
- **`examples/ensemble/`** — Demonstrates parameter sweeps via `jax.vmap` over B1 values.

### Conventions

- All dataclasses holding JAX arrays must be registered with `@jax.tree_util.register_dataclass`
- Fluxes are positive upward (w_th > 0 = upward heat flux)
- Units are SI throughout
- Finite differences are 1st-order on the staggered grid; diffusion terms live on half levels
- The closure applies a 1-2-1 filter to both length scales (L) and eddy diffusivities to suppress vertical oscillations; the length-scale filter specifically prevents L_B = q/N from amplifying numerical noise above the boundary layer
- Surface TKE boundary condition: `qke_sfc = B1^(2/3) * u_st²`, where B1=24.0 by default
- Tests run in x64 precision (JAX default is x32) to match validation results; enforce via `jax.config.update("jax_enable_x64", True)` in test fixtures
- Partial condensation and level-3 closure are not implemented

---
> Source: [mpierzyna/jax_scm](https://github.com/mpierzyna/jax_scm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
