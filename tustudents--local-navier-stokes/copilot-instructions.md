## local-navier-stokes

> This file helps the Gemini agent understand the project's context, conventions, and goals.

# Gemini Project Configuration

This file helps the Gemini agent understand the project's context, conventions, and goals.

## Project Overview

*   **Name:** Local-Navier-Stokes
*   **Description:** This project explores the Local Navier-Stokes (LNS) equations, a novel approach to fluid dynamics. It includes a series of Jupyter notebooks that build from foundational concepts to advanced applications, including relativistic fluid dynamics.
*   **Keywords:** fluid dynamics, Navier-Stokes, LNS, computational fluid dynamics (CFD), Jupyter notebooks, Python

## Conventions

*   **Languages:** Python
*   **Notebooks:** The project is structured as a series of Jupyter notebooks. Each notebook should be self-contained and clearly documented. When modifying notebooks, strictly adhere to their JSON formatting, including proper handling of escape characters and line endings. Pay special attention to the correct rendering of LaTeX-style formulas within markdown and code cells.
*   **Dependencies:** Project dependencies are managed with `uv` and are listed in `pyproject.toml`.
*   **Code Style and Formatting:** Adhere strictly to existing code style and formatting conventions within files. Preserve indentation, line endings, and general code structure when making modifications.

## Goals

The primary goal of this project is to develop and validate the LNS framework. This includes:
1.  Demonstrating the advantages of LNS over traditional Navier-Stokes equations.
2.  Implementing LNS solvers for various physical scenarios.
3.  Exploring the application of LNS to complex and relativistic fluids.

## File Structure

```
- 00_Motivation
  - 00a_NS_Critique.ipynb
  - 00b_NS_Critique.ipynb
  - 00c_NS_Critique.ipynb
  - 01_LNS_Solver_1D_Toy model.ipynb
- 01_Foundation.md
- 01_LNS
  - 01_LNS_Solver_1D_EnhancedPhysics.ipynb
  - 02_LNS_Transonic_Analysis.ipynb
- 03_LNS_Series2
  - LNS_Series2_NB1_AdvancedNumerics.ipynb
  - LNS_Series2_NB2_3D_Implementation.ipynb
  - LNS_Series2_NB3_Transition.ipynb
  - LNS_Series2_NB4_Developed_Turbulence.ipynb
- 04_LNS_for_Complex_Fluids
  - outline.md
- 05_From_LNS_to Einstein's_Universe
  - Relativistic_Series_NB1_Classical_Failure.ipynb
  - Relativistic_Series_NB2_Israel_Stewart.ipynb
  - Relativistic_Series_NB3_Applications.ipynb
- CLAUDE.md
- GEMINI.md
- pyproject.toml
- uv.lock
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TUstudents)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TUstudents)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
