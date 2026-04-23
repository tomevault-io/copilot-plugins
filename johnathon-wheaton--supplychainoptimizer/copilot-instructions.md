## supplychainoptimizer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Supply Chain Optimizer is a mixed integer linear programming (MILP) solver designed for complex network flow optimization problems. It's built for operations research practitioners and supports multi-scenario analysis, multi-objective optimization, and multi-modal transportation networks.

## Development Commands

### Running the Application
```bash
# Run optimization with Excel input
python src/main.py src/examples/demo_inputs_transportation_facility_location.xlsx -o output_results.xlsx

# Run optimization with JSON input
python src/main.py src/examples/demo_inputs_transportation_facility_location.json -o output_results.xlsx

# Run with custom log level
python src/main.py input_file.json -o output.xlsx --log-level DEBUG
```

### Installation and Setup
```bash
pip install -r requirements.txt
```

### Project Structure Notes
- No build, test, or lint commands are configured - the project uses direct Python execution
- Dependencies are managed via requirements.txt (numpy, pandas, PuLP, PyYAML, Dash components)
- The project includes pre-compiled solver binaries in `src/solvers/` (HiGHS, CBC, SCIP)

## Architecture Overview

### Core Components

**Entry Point**: `src/main.py`
- Main optimization pipeline orchestration
- Command-line argument parsing
- File I/O coordination
- Multi-scenario processing loop

**Data Layer** (`src/data/`):
- **Readers**: Factory pattern for Excel/JSON input parsing (`readers/reader_factory.py`)
- **Writers**: Excel output generation (`writers/excel_writer.py`)  
- **Processors**: Data transformation and scenario handling (`processors/`)
- **Preprocessors**: Input data validation and normalization (`preprocessors/`)

**Models** (`src/models/`):
- **Network**: Core network representation and set generation (`network.py`)
- **Node**: Individual node modeling (`node.py`)

**Optimization Engine** (`src/optimization/`):
- **Constraints**: Modular constraint system with base class pattern (`constraints/`)
  - Flow, capacity, transportation, resource, age, and cost constraints
- **Variables**: Decision variable creation (`variables/variable_creator.py`)
- **Objectives**: Multi-stage and multi-objective handling (`objectives/objective_handler.py`)
- **Solvers**: MILP solver interfaces (`solvers/milp_solver.py`)

**Configuration** (`src/config/`):
- **Settings**: Comprehensive configuration system supporting JSON/YAML (`settings.py`)
- Solver settings, network parameters, resource constraints, logging configuration

**Utilities** (`src/utils/`):
- Logging utilities with rotation support
- Excel to JSON conversion tools

### Key Design Patterns

**Multi-Scenario Processing**: The system processes scenarios sequentially, filtering input dataframes by scenario column and supporting wildcard scenarios (`*`)

**Constraint Architecture**: Each constraint type inherits from `BaseConstraint` and implements a `build(model)` method to add constraints to the PuLP model

**Factory Pattern**: Reader creation uses factory pattern to support multiple input formats (Excel, JSON)

**Configuration Management**: Hierarchical settings system with validation, supporting both programmatic and file-based configuration

### Data Flow

1. **Input Processing**: Factory creates appropriate reader → Data validation and preprocessing → Scenario splitting
2. **Model Building**: Network creation → Parameter generation → Variable creation → Constraint building  
3. **Optimization**: Multi-stage objective solving → Solver execution (HiGHS/CBC/SCIP)
4. **Output Processing**: Results processing → Excel export with multiple sheets

### Solver Integration

The system supports three solvers with configurable paths:
- **HiGHS** (default): `src/solvers/highs.exe`
- **CBC**: `src/solvers/Cbc-master-x86_64-w64-mingw32/bin/cbc.exe`
- **SCIP**: `src/solvers/SCIPOptSuite-9.2.1-win64.exe`

Solver selection and parameters are managed through the Settings system.

### Input Data Format

Input files (Excel/JSON) must follow a specific schema documented in `docs/Input_Instructions.md`. Key sheets include:
- Parameters, Scenarios, Objectives, Periods, Products
- Nodes, Node Types, Node Groups  
- Flow, Transportation, Capacity, and Cost definitions
- Supports wildcard notation (`*` for "all", `@` for global wildcard)

### Visualization

The project includes a Dash-based visualization dashboard (`src/results/visualization_dashboard.py`) for results analysis.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Johnathon-Wheaton) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
