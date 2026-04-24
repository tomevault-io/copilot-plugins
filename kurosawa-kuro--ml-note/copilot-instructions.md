## ml-note

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a machine learning study repository focused on data analysis and experimentation. The repository uses Python with Jupyter Lab for interactive development and contains a DuckDB database for dataset storage.

## Development Environment

### Python Setup
- **Python Version**: 3.12.3 (located at `/usr/bin/python3`)
- **Package Manager**: pipx for isolated installations
- **Primary Tool**: Jupyter Lab for interactive development

### Required Tools
- pipx: `sudo apt install pipx`
- Jupyter Lab: `pipx install jupyterlab`

## Common Commands

### Starting Development Environment
```bash
jupyter-lab
```
This starts Jupyter Lab server for interactive Python development and data analysis.

### Environment Check
```bash
python3 --version  # Verify Python 3.12.3
which jupyter-lab  # Should be at /home/wsl/.local/bin/jupyter-lab
```

## Architecture and Data

### Repository Structure
- `README.md`: Basic setup instructions in Japanese
- `kaggle_datasets.duckdb`: DuckDB database file (version 64) containing datasets for analysis
- `.claude/`: Claude Code configuration directory

### Data Storage
The repository uses DuckDB for efficient storage and querying of datasets. The main database file `kaggle_datasets.duckdb` contains the working datasets for machine learning experiments.

### Development Approach
This is a study/research repository focused on:
- Interactive data analysis using Jupyter notebooks
- Machine learning experimentation
- Dataset exploration and manipulation through DuckDB

## Working with Data
When working with datasets, interact with the DuckDB database file rather than creating separate data files. This approach maintains data consistency and leverages DuckDB's analytical capabilities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kurosawa-kuro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
