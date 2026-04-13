## resram-ng

> This project implements the theoretical framework described in the provided publications to benchmark DFT exchange-correlation functionals (XCFs) against experimental **Femtosecond Stimulated Raman Spectroscopy (FSRS)** data of Bodipy.

# Project Understanding: Bodipy FSRS Analysis

This project implements the theoretical framework described in the provided publications to benchmark DFT exchange-correlation functionals (XCFs) against experimental **Femtosecond Stimulated Raman Spectroscopy (FSRS)** data of Bodipy.

## Core Components and Mapping to Theory

1.  **Theoretical Model**: The code in `resram_core.py` implements the **Independent Mode Displaced Harmonic Oscillator (IMDHO)** formalism.
    *   **Absorption/Fluorescence**: Calculated using the time-correlator $A(t)$ (Equation S6 in SI) and the Brownian oscillator line shape function $g(t)$ (Equation S9).
    *   **Resonance Raman Cross Sections**: Implemented in `cross_sections(obj)` using the first-order approximation (Equation S5).

2.  **Fitting Procedure**: Optimized via `lmfit` in `FSRSanalysis_v2.ipynb` or the `ResRamQt.py` GUI. It minimizes the residual between experimental and calculated spectra by adjusting:
    *   **Dimensionless displacements ($\Delta_k$)** for each vibrational mode.
    *   **Global Parameters**: `gamma` ($\Gamma$, homogeneous broadening), `theta` ($\theta$, inhomogeneous broadening), `E0` (vertical transition energy), and `transition_length` ($M$).

3.  **Performance Optimization**:
    *   **Rust Backend (`resram_rust`)**: An optional high-performance C++-equivalent extension that provides a **~4x speedup** for integration-heavy calculations.
    *   **State Consistency**: The `load_input.update_params()` method in `resram_core.py` ensures that all derived physical constants and integration grids are consistently refreshed across the Python and Rust engines.

4.  **Graphical User Interface (`ResRamQt.py`)**:
    *   **Asynchronous Processing**: Uses `QThreadPool` to offload heavy spectroscopic calculations to background threads, maintaining a smooth 60fps UI.
    *   **Debounced Interactions**: Parameter changes are debounced (300ms) to prevent redundant calculation cycles during rapid user input.
    *   **Interactive Control**: Provides checkboxes to toggle visibility of vibrational modes and selective fitting of parameters.

5.  **CI/CD and Deployment**:
    *   **GitHub Actions**: Automates the compilation of `resram_rust` and the `ResRamQt` standalone executables (via Nuitka) for Windows, Linux, and macOS.

## Purpose

The project effectively maps the multidimensional excited-state potential energy surface (PES) by determining the displacements that best reproduce the vibronic features of the experimental spectra. This allows for a direct comparison with TD-DFT predictions, helping identify which exchange-correlation functionals best describe the electronic excited state of Bodipy.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alchemist-Aloha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Alchemist-Aloha)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
