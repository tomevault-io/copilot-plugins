## bess-experiments

> This file provides guidance to Gemini Code (Gemini.ai/code) when working with code in this repository.

# Gemini.md

This file provides guidance to Gemini Code (Gemini.ai/code) when working with code in this repository.

## Development Commands

### IMPORTANT: Virtual Environment Usage
**Gemini Code MUST ALWAYS use the virtual environment for all Python commands.**
All Python-related commands (pip, python, etc.) must be prefixed with `source venv/bin/activate &&`

### Environment Setup
```bash
# Create virtual environment (first time only)
python -m venv venv

# Activate virtual environment and install dependencies
source venv/bin/activate && pip install -r project_requirements.txt

# Install system solver dependencies (Fedora/RHEL)
sudo dnf install python3-pip coin-cbc

# Install system solver dependencies (Ubuntu/Debian)  
sudo apt install python3-pip coinor-cbc

# Test solver availability (always use venv)
source venv/bin/activate && python test_solvers.py
```

### Running the Application
```bash
# Run main optimization demo (always use venv)
source venv/bin/activate && python local_run.py

# Run all example scenarios (always use venv)
source venv/bin/activate && python examples.py

# Run experiment script (always use venv)
source venv/bin/activate && python experiment.py

# Run web service (always use venv)
source venv/bin/activate && python app.py

# Test web service (always use venv)
source venv/bin/activate && python test_api.py
```

### Makefile Commands
```bash
# Activate virtual environment and install requirements
make start-env

# Freeze current dependencies to requirements.txt
make freeze-deps
```

## Architecture Overview

This is a **Battery Energy Storage System (BESS) maintenance optimization** project that uses Mixed Integer Linear Programming (MILP) to find optimal maintenance scheduling windows.

### Core Components

1. **MaintenanceOptimizer Class** (`local_run.py`)
   - Main optimization engine using PuLP library
   - Supports multiple maintenance events with different durations
   - Optimizes both electricity costs and labor costs
   - Uses CBC solver for MILP optimization
   - Generates visualizations of results

2. **Key Optimization Features:**
   - Time discretization (1-hour slots by default)
   - Binary decision variables for maintenance scheduling
   - Constraint handling for non-overlapping maintenance events
   - Cost minimization objective function combining electricity and labor costs

3. **Example Scenarios** (`examples.py`)
   - Basic optimization with default settings
   - High failure risk scenarios
   - Volatile electricity price patterns
   - Different planning horizons (12h, 24h, 48h)
   - Equipment with varying power consumption

### Dependencies and Solvers

- **Required Solver**: CBC (COIN-OR Branch and Cut) for MILP optimization
- **Primary Libraries**: PuLP (optimization), NumPy, Matplotlib (visualization)
- **Alternative Solvers**: GLPK, Gurobi, CPLEX (if available)

### Mathematical Model

The optimizer implements a MILP formulation with:
- **Variables**: Binary start/active variables for each maintenance event and time slot
- **Objective**: Minimize combined electricity costs and labor costs
- **Constraints**: Single start per event, duration linking, timeline limits, non-overlap between events

### File Structure

- `local_run.py` - Main optimization implementation and demo
- `examples.py` - Multiple scenario demonstrations  
- `test_solvers.py` - Solver availability checker
- `experiment.py` - Additional experimentation script
- `app.py` - Flask web service wrapper for the optimization model
- `test_api.py` - Web service testing script
- `project_requirements.txt` - Core Python dependencies
- `requirements.txt` - Full system package list (Fedora-specific)
- `venv/` - Python virtual environment (isolated dependencies)

### Key Configuration

- Planning horizon: Configurable (default 24-48 hours)
- Time slots: 1-hour intervals
- Maintenance durations: Configurable per event (1-3 hours typical)
- Cost patterns: Customizable electricity and labor cost dictionaries

The system is designed for educational/research purposes to demonstrate optimization techniques for energy system maintenance scheduling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akantsevoi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akantsevoi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
