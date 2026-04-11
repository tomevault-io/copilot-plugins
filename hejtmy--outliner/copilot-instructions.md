## outliner

> **outliner** is an R package designed to visualize data processing pipelines. It solves the problem of "black box" functions by parsing R scripts and function definitions to generate an interactive node-graph visualization of the data flow.

# Project Context: outliner

## Purpose
**outliner** is an R package designed to visualize data processing pipelines. It solves the problem of "black box" functions by parsing R scripts and function definitions to generate an interactive node-graph visualization of the data flow.

## Project Structure
The project is a hybrid R package and React application.

- **Root**: Standard R package structure (`DESCRIPTION`, `NAMESPACE`).
- **`R/`**: Backend logic.
  - `parser_meta.R`: Extracts metadata (name, description) from custom string syntax inside functions.
  - `parser_flow.R`: Parses R scripts to detect assignments and pipe connections.
  - `visualizer.R`: Prepares graph data and serves the React app via `httpuv`.
- **`inst/app/`**: Frontend source code (React + Vite).
  - Use `npm run dev` for local dev.
  - Use `npm run build` to compile.
- **`inst/www/`**: Compiled frontend assets (HTML/JS/CSS). The R package serves these files.
- **`vignettes/repro_example/`**: A self-contained example project for verification.
- **`verify_script.R`**: Automation script to run the example.

## Current Status & Workflow
- **State**: Active development.
- **Stack**: R (backend), React/Vite (frontend).
- **Workflow**:
  1.  Modify R code -> `devtools::load_all()`.
  2.  Modify Frontend (`inst/app`) -> `npm run build` -> Copy `dist/` to `inst/www/`.
  3.  Verify -> Run `verify_script.R`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hejtmy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hejtmy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
