## engineering-fundamentals

> This repository contains a collection of Python examples and Jupyter notebooks demonstrating practical applications across various engineering disciplines. It primarily focuses on structural engineering, finite element analysis, and data handling, serving as a companion repository for the Flocode Engineering Insights newsletter.

# Project Overview

This repository contains a collection of Python examples and Jupyter notebooks demonstrating practical applications across various engineering disciplines. It primarily focuses on structural engineering, finite element analysis, and data handling, serving as a companion repository for the Flocode Engineering Insights newsletter.

# Architecture & Technologies

- **Language**: Python (>=3.11).
- **Environment & Dependency Management**: uv is used for managing dependencies and environments across isolated sub-projects using standard PEP 621 `pyproject.toml` files.
- **Key Libraries**:
  - **Data & Mathematics**: `pandas`, `numpy`, `scipy`.
  - **Visualization**: `matplotlib`, `seaborn`, `vtk`.
  - **Engineering & Structural**: `PyNiteFEA` (3D static and dynamic finite element analysis), `symbeam` (analytical beam bending solutions).
- **Repository Structure**:
  - `Pandas/`: Demonstrates data manipulation for civil/structural engineers using Pandas.
  - `SQL/`: Contains SQL examples and tutorials in Jupyter notebooks.
  - `Structural/`: Core folder for structural engineering scripts, notebooks (dynamics, matrices, beam formulas), and sub-projects.
  - `Structural/Pynite/`: Specific FEA examples utilizing the `PyNiteFEA` library (e.g., space frames, hydrostatic loads).
  - `Structural/SAP2000/`: Files and scripts relating to the CSI SAP2000 API.
  - `Structural/symbeam/`: Examples for generating bending moment and shear force diagrams using the `symbeam` library.

# Building and Running

As a collection of standalone examples and tutorials rather than a unified application, execution is done on a per-folder basis.

To run scripts or notebooks within a specific directory:
1. Navigate to the desired subdirectory (e.g., `cd Structural/Pynite`).
2. Sync the environment and dependencies using uv:
   ```bash
   uv sync
   ```
3. Run the target script or launch a Jupyter notebook environment:
   ```bash
   uv run python <script_name>.py
   # OR
   uv run jupyter notebook
   ```

# Development Conventions

- **Isolation**: Each major topic or library example should maintain its own directory with a dedicated `pyproject.toml` file to manage its specific dependencies via uv.
- **Purpose**: Code should be illustrative and educational, aimed at readers of the Flocode newsletter.
- **Licensing**: All contributions and existing code are provided under the MIT License.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joreilly86)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joreilly86)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
