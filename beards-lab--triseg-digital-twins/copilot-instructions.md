## triseg-digital-twins

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MATLAB codebase supporting the manuscript *"Identification of Digital Twins to Guide Interpretable AI for Diagnosis and Prognosis in Heart Failure"* (Feng et al.). It identifies patient-specific digital twins of cardiovascular physiology by fitting the **TriSeg** biventricular heart model + lumped-parameter circulation to clinical measurements (echocardiography, RHC, CMR) of heart-failure patients.

There is no build system, no test suite, and no package manifest. Everything runs inside MATLAB (`ode15s`, Optimization Toolbox, Global Optimization Toolbox, Statistics and Machine Learning Toolbox, Parallel Computing Toolbox for `UseParallel`).

## Running the code

Open MATLAB in the repo root. The two canonical entry points are sections of [Driver.m](Driver.m):

- **Section 1 — canonical male/female subject**: set `GENDER` and `Sim` (1 = simulate from saved modifiers, else run `Srdopt` to optimize). Reads `modifiers_male.csv` / `modifiers_female.csv`.
- **Section 2 — heart-failure patients** (`load AllPatients.mat`): loop over `PATIENT_NO` (1..370 for UM cohort, example patient in paper is 192). Set `RUNOPT = 0` to load a precomputed fit from `SimsUMFinal/P_NO<n>.mat`, or `RUNOPT = 1` to launch [HFopt.m](HFopt.m) (GA → patternsearch → fminsearch). `MRI_flag` toggles whether CMR-derived targets are included.

Plotting / inspection scripts (run after `runSim`): [NplotFit.m](NplotFit.m) (HF 6-panel), [NplotSrd.m](NplotSrd.m) (canonical 4-panel), [GetMovie.m](GetMovie.m), [See_TriSeg.m](See_TriSeg.m).

There is no "single test" — the closest equivalent is running one patient via Driver.m Section 2 with `RUNOPT=0` and `print_sim=true` (uncomment in [runSim.m](runSim.m) area; the cost-printing block at the bottom of runSim emits per-target percent error and weighted cost).

## Architecture

The pipeline is a **script-based** dataflow where workspace variables (`targets`, `inputs`, `mods`, `params`, `init`, `t`, `y`, `o`) are passed implicitly between MATLAB scripts. This is not a function-based API — many "steps" are scripts that read/write the caller's workspace. Be careful when refactoring: renaming a workspace variable in one script silently breaks downstream scripts.

### Data flow

```
targetVals_*.m  ──►  estiminiParams.m  ──►  optParams.m  ──►  runSim.m  ──►  Nplot*/GetMovie/See_TriSeg
   (clinical                (initial            (apply           (ode15s on
    targets +                params +            modifier         dXdTDAE/dXdTode +
    inputs +                 initial             vector m)        post-processing +
    mods list)               state)                               cost function)
```

1. **`targetVals_male.m` / `targetVals_female.m` / `targetVals_HF.m`** — return `targets` (clinical measurements to fit), `inputs` (sex, height, weight, HR, etc.), and `mods` (the list of parameter names that are adjustable for this patient). `targetVals_HF.m` also returns `Windowdate` and takes `(patients, PATIENT_NO, ModelWin, MRI_flag)`.
2. **`estiminiParams.m`** — produces `INIparams` (a parameter struct seeded from targets/inputs via allometric and physiologic relations) and `INIinit` (initial state for the ODE). Internally calls **`geom_0.m`** (wall/lumen volumes) and **`calc_xm_ym.m`** (TriSeg geometry: solves for `xm_LV/SEP/RV` and `ym` so initial conditions are mechanically consistent).
3. **`optParams.m`** — applies a modifier vector `m` (multiplicative scalings of the entries in `mods`) to `INIparams`/`INIinit`, returning the `params`/`init` actually used by the ODE.
4. **`dXdTDAE.m`** (DAE form, mass matrix) and **`dXdTode.m`** (pure ODE fallback) — define the right-hand side of the cardiovascular state equations (TriSeg geometry + sarcomere mechanics + 0-D circulation). [runSim.m](runSim.m) tries the DAE form first and falls back to the ODE form on failure (the two have **different state orderings and different output row layouts** — see the two parallel `y(:,k)` / `o(k,:)` blocks at lines ~45–117 and ~145–248). Any change to one form must be mirrored in the other.
5. **`runSim.m`** — solves with `ode15s` (custom timeout via `odeWithTimeout1/2` nested at the bottom), keeps the last 2 cardiac cycles as steady-state, post-processes valve flows / regurgitation / stenosis grades / chamber dimensions / pressures, then computes a **weighted, calibrated, normalized squared-error cost** of `o_vals` vs `targets` plus a `tax` term (sarcomere length, E/A ratio, and — when `MRI_flag==0` — empirical RV passive/active stress constraints loaded from `Kconstrain.mat`). The cost printout uses the symbolic exchange rate `EX = 7.1720` (USD→CNY on 2025-06-24) as a fixed scaling — do not "clean this up", it is a deliberate constant.
6. **`evaluateModel.m` / `evaluateModelUmich.m`** — wrap the above into a single `cost = f(m, ...)` that the optimizers in `HFopt.m` / `Srdopt.m` call. `HFopt.m` uses GA → patternsearch (with `searchga` hybrid) → `fminsearch`, iterated until improvement < 10.
7. **Bounds**: [m_bounds.m](m_bounds.m) returns `[ub, lb]` for the modifier vector based on the names in `mods`.

### Saved-result conventions

Optimization runs on the cluster (Great Lakes / supercomputer nodes) save per-patient `.mat` files into `SimsUMFinal/P_NO<n>.mat` (UM cohort, 1–370) and `SimsUWFinal/P_NO<n>.mat` (UW cohort, 1–112). Each `output` struct contains `mods`, `m`, `params`, `init`, `targets`, `inputs`, and (when `runSimOnGL` succeeded) `o`, `y`, `t`. `Driver.m` Section 2 with `RUNOPT=0` loads these directly instead of re-running optimization. New optimization in HFopt currently writes to `Sims0707UM/` — when adding a new run, mirror that naming or update both Driver and HFopt together.

### TriSeg state-variable cheat sheet

DAE form (`dXdTDAE`): `y = [xm_LV, xm_SEP, xm_RV, ym, Lsc_LV, Lsc_SEP, Lsc_RV, V_LV, V_RV, V_SA, V_SV, V_PA, V_PV]` — first 4 are algebraic (mass-matrix zeros).
ODE form (`dXdTode`): `y = [Lsc_LV, Lsc_SEP, Lsc_RV, V_LV, V_RV, V_SA, V_SV, V_PA, V_PV, V_LA, V_RA]`, with `xm_*` and `ym` recovered from `iniGeo` + algebraic solve inside the RHS.

## Things to know before editing

- **Workspace coupling**: scripts (not functions) like `runSim`, `HFopt`, `Srdopt`, `NplotFit`, `GetMovie`, `See_TriSeg` rely on caller-scope variables. Converting them to functions is a non-trivial refactor — do not do it incidentally.
- **`MRI_flag`** gates whether CMR targets (`Hed_LW/SW/RW`, `LV_m`, `RV_m`, `LVIDd/s`, `RVIDd/s`) are in `targets` *and* whether the empirical RV-from-LV constraint (`Kconstrain.mat` regression) is added to the cost. Both call sites must agree.
- **Patient data is PHI-adjacent**. `AllPatients.mat`, `MRN.xlsx`, `RawPatientInfo1.xlsx`, `cMRI.xlsx`, `UMUWP.csv` contain de-identified clinical data tied to UM/UW cohorts. Do not commit derived files that re-introduce identifiers, and do not paste their contents into external services.
- **`.asv` files** (`Driver.asv`, `See_TriSeg.asv`) are MATLAB autosave artifacts — ignore them; do not edit.
- The cost-function weighting block in `runSim.m` (lines ~941–1160) is hand-tuned per-target and per-cohort; changing a single weight cascades through every optimization result. Treat it as load-bearing configuration, not "magic numbers to clean up."
- Comments inside [runSim.m](runSim.m) and [HFopt.m](HFopt.m) include disabled "kill" branches (e.g. `error('unreal condition')`). They are intentionally commented; the author iterates between strict and permissive constraints — confirm before re-enabling.

## README

The repo's [README.md](README.md) gives the user-facing function descriptions (Source Data, Functions, User Guide). It is the right reference when a user asks "what does *X*.m do?"; this file complements it with architectural detail not in the README.

---
> Source: [beards-lab/TriSeg-Digital-Twins](https://github.com/beards-lab/TriSeg-Digital-Twins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
