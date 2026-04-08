## uber-spatial-pricing

> Industrial organization paper scaffold for studying spatial pricing, demand, and market structure in Chicago using Uber-style pricing data plus neighborhood and transit inputs.

# Uber Spatial Pricing - Agent Guidelines

## Project Overview
Industrial organization paper scaffold for studying spatial pricing, demand, and market structure in Chicago using Uber-style pricing data plus neighborhood and transit inputs.

## Canonical Project Reference
- `uber_spatial_pricing.md` is the canonical research master document for this project
- Treat it as the ground-truth reference for research design, empirical priorities, paper framing, and task sequencing whenever those choices are ambiguous

## Data Analysis Workflow
- Task-based workflow identical to the spatial project pattern used elsewhere
- Every task has a dedicated folder in `tasks/` with `code/`, `input/`, and `output/` subfolders
- Every task also has a `temp/` folder created through shared make rules
- Each task has its own Makefile
- Tasks use symlinking to connect upstream outputs to downstream inputs
- Never use `mkdir` explicitly in a task makefile; rely on `../../generic.make` to create directories for all task makefiles, and call it in every single task makefile after the `all` target
- Prefer `R` over `Python` for new analysis, data-prep, estimation, and mapping tasks
- Use `Python` only when there is a clear technical reason to do so, and still follow the same task and make conventions
- Avoid ad hoc root-level scripts; new work should live inside a task with a task-specific Makefile

## Project Structure
- `tasks/` - Analysis tasks, data prep tasks, and model-building tasks
- `paper/` - Minimal paper scaffold and future writeup
- `tasks/*/code/` - Task entry scripts and Makefiles

## Running Tasks
- Always execute tasks by running `make` from the `code` folder within any task
- All paths should be relative to the code folder
- Run the full project from the root with `make` or `make tasks`

## Key Files
- `tasks/shell_functions.make` - Make helpers for `R` and `Python` execution
- `tasks/shell_functions.sh` - Shared shell functions
- `tasks/generic.make` - Common task directory rules
- `tasks/setup_environment/code/packages.R` - R package bootstrap (source this in all R scripts)
- `tasks/setup_environment/code/requirements.txt` - Python package reference for bootstrap checks

## Research Design
- Focus on Chicago spatial pricing rather than the Red Line welfare model
- Build reusable geographic and transit inputs before ingesting proprietary or licensed pricing data
- Organize empirical work as separate tasks for data preparation, panel construction, descriptive outputs, and baseline estimation
- Prefer outputs that are easy to inspect and reuse across tasks: `csv`, `geojson`, `parquet`, `rds`, and simple figures

## Data Sources
1. Chicago tract and city boundaries via Census or local GIS sources
2. CTA transit network inputs and stop locations
3. Uber-style trip or pricing microdata supplied separately by the researcher
4. Derived crosswalks between geographic units, stops, and market definitions

## Initial Empirical Goal
Use Chicago spatial and transit data to build a reproducible pricing panel that can support IO-style descriptive analysis and a baseline estimating specification before the substantive paper design is finalized.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JacobHerbstman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JacobHerbstman)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
