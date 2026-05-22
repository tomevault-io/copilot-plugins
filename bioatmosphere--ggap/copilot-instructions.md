## ggap

> Generates 4 plot types: forest_dynamics, soil_biogeochemistry, environmental_conditions, summary_dashboard.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GGap is a GPU-accelerated agent-based forest gap dynamics model. It combines the UVAFME (University of Virginia Forest Model Enhanced) forest simulation with the SAGESim agent-based modeling framework to create a scalable, GPU-enabled forest Gap model.

The project integrates three major components:
1. **SAGESim** - GPU-accelerated agent-based modeling framework using CuPy and MPI
2. **UVAFME** - Traditional forest gap model (Python translation from Fortran)
3. **GGap** - Integration layer combining UVAFME processes with SAGESim's agent framework

## Development Environment Setup

### Dependencies

Python 3.13+ is required. Install dependencies using uv:

```bash
uv sync
```

Key dependencies:
- `cupy` - GPU array operations (requires CUDA or ROCm)
- `mpi4py` - Parallel execution across multiple ranks
- `numpy` - Numerical operations
- `rasterio` - Geospatial data handling
- `earthengine-api` & `geemap` - NLCD forest data access

### GPU Requirements

- **NVIDIA GPU**: CUDA drivers required
- **AMD GPU**: ROCm 5.7.1+ required

Install CuPy and mpi4py according to your system's hardware before installing other dependencies.

## Running the Model

### Quick Demo (No GPU Required)

Show available species and model architecture:

```bash
uv run python main.py
```

Or with standard Python:
```bash
python main.py
```

This demo doesn't require GPU/CuPy - it just displays species data.

### GGap Single-Site Simulation (GPU Required)

The primary entry point runs a single-site simulation with CSV-initialized species and climate from UVAFME input files:

```bash
cd gap
python run_one_site.py --num_gaps 200 --pool_size 1000 --years 500
```

Command-line options:
- `--num_gaps`: Number of gaps per site (default: 200)
- `--pool_size`: Max tree slots per gap (default: 1000)
- `--years`: Simulation duration in years (default: 1000)
- `--report_interval`: Years between progress reports and CSV output (default: 10)
- `--site_id`: Site ID from UVAFME CSV files (default: 0)
- `--data_dir`: Directory containing UVAFME CSV files (default: input_data)
- `--prefix`: File prefix for UVAFME CSV files (default: UVAFME2012)
- `--output_dir`: Directory for CSV output files (default: output_data)
- `--no_tree_data`: Skip writing tree_data.csv (can be very large)

**Example runs:**
```bash
# Quick test (10 gaps, 50 years)
python run_one_site.py --num_gaps 10 --years 50

# Full simulation
python run_one_site.py --num_gaps 200 --pool_size 1000 --years 1000

# Different site
python run_one_site.py --site_id 1 --data_dir input_data --prefix UVAFME2012
```

### Plotting Output

After a simulation, generate plots from CSV output:

```bash
cd gap
python plot_outputs.py --output-dir ../output_data --format png
```

Options: `--plots-dir`, `--format` (png/pdf/svg), `--dpi`, `--style`, `--show/--no-show`

Generates 4 plot types: forest_dynamics, soil_biogeochemistry, environmental_conditions, summary_dashboard.

### SAGESim Examples (Reference)

SAGESim provides reference implementations in `SAGESim/examples/`:

**SIR Epidemic Model:**
```bash
cd SAGESim/examples/sir
mpirun -n 4 python run.py --num_agents 10000 --percent_init_connections 0.1 --num_nodes 1
```

**Forest Gap Model (SAGESim Reference):**
```bash
cd SAGESim/examples/forest_gap
mpirun -n 4 python run.py --num_trees 500 --forest_size 100 --years 100
```

### UVAFME Model (Original)

Run the standalone UVAFME forest model:

```bash
cd UVAFME
python main.py
```

### Testing

Test model initialization (no GPU):
```bash
python -c "from gap.gap_model import GAPModel; m = GAPModel(); s = m.initialize_site(); print(s['site_name'], len(s['species']), 'species')"
```

Run SAGESim tests:
```bash
cd SAGESim
pytest tests/
```

## Code Architecture

### SAGESim Framework (`SAGESim/sagesim/`)

SAGESim is a GPU-accelerated agent-based modeling framework with double-buffering for race condition prevention.

**Core Components:**
- `model.py` - Main Model class orchestrating simulation execution
- `breed.py` - Agent breed definitions with properties and step functions
- `agent.py` - AgentFactory for creating and managing agents
- `space.py` - Spatial structures (NetworkSpace, GridSpace)
- `utils.py` - GPU kernel utilities for agent data access

**Key Concepts:**

1. **Breeds**: Define agent types with properties and step functions
   - Register properties: `self.register_property(name, default_value)`
   - Register step functions: `self.register_step_func(func, filepath, priority)`

2. **Step Functions**: GPU kernels decorated with `@jit.rawkernel(device="cuda")`
   - Must follow strict CuPy JIT constraints (see CuPy Limitations below)
   - Executed in parallel across all agents on GPU
   - Use double-buffering pattern for race-free writes

3. **Model Execution Flow**:
   - `model.setup(use_gpu=True)` - Initialize breeds, analyze step functions, generate GPU kernels
   - `model.simulate(ticks, sync_workers_every_n_ticks)` - Run simulation for N ticks
   - MPI synchronization occurs every N ticks across workers

4. **Double Buffering**: SAGESim automatically analyzes step functions to detect write operations and creates separate write buffers to prevent race conditions when multiple agents modify shared data.

### UVAFME Model (`UVAFME/vegetation/`)

UVAFME is a traditional forest gap model translated from Fortran to Python.

**Core Modules:**
- `uvafme.py` - Main model orchestration and site loop processing
- `species.py` - Tree species data and response functions (temperature, light, drought, fire, flood)
- `site.py` - Site-level climate and species management
- `soil.py` - Soil biogeochemistry (decomposition, water cycle, 3-layer structure)
- `climate.py` - Climate data processing and conversions
- `tree.py` - Individual tree growth and mortality
- `plot.py` - Plot-level processes and canopy interactions

**Key Processes:**
- Species response functions (parabolic temperature, exponential drought/light)
- Soil carbon and nitrogen cycling across A0, A, and Base layers
- Canopy light interactions and shading
- Tree growth, mortality, and recruitment

### GGap Integration (`gap/`)

GGap implements the full UVAFME forest dynamics using a 3-level agent hierarchy (Site → Gap → Tree) with 7-priority step functions running on GPU.

**Agent Hierarchy:**
- **Site agents** - Hold soil pools (A0/A/Base C/N), climate data (12 months × 4 variables), available nitrogen
- **Gap agents** - Aggregate litter/demand from trees, compute light profiles, manage seedbank/renewal
- **Tree agents** - Individual trees with species parameters, growth state, biomass tracking

**Core Files:**
- `gap_model.py` - GAPModel class: breed registration, CSV-based site/species initialization, gap/tree creation, data collection
- `run_one_site.py` - Main simulation runner with GAPpy-compatible CSV output
- `output_utils.py` - OutputWriter producing 5 CSV files (site_data, soil_data, genus_data, species_data, tree_data)
- `plot_outputs.py` - Visualization of simulation outputs (4 plot types)
- `step_func_code.py` - Generated GPU kernel code

**Step Functions** (`step_functions/`):
- `gap/gap_litter_aggregate_step.py` (P0) - Aggregate litter from trees, density-based recruitment count
- `site/soil_step.py` (P1) - Daily soil biogeochemistry (365 days), climate variability, water balance
- `tree/tree_potential_growth_step.py` (P2) - Environmental responses, potential diameter growth
- `gap/gap_demand_aggregate_step.py` (P3) - Sum N demand across trees per gap
- `site/site_nutrient_step.py` (P4) - Compute N supply ratio
- `gap/gap_sync_step.py` (P5) - Relay climate and N ratio to trees, clear accumulators
- `tree/tree_actual_growth_step.py` (P6) - N-limited growth, mortality, biomass update, renewal

**Species Data:**
- 32 species loaded from `input_data/UVAFME2012_specieslist.csv` (filtered by site range)
- 20 genera (Acer, Betula, Carya, Fagus, Fraxinus, etc.)
- Species parameters: max age/diam/ht, growth rate, shade/drought/flood tolerance, degree-day range, leaf area, wood density

**Output System:**
- 5 GAPpy-compatible CSV files with proper scaling (plotscale = HEC_TO_M2/plotsize)
- Per-gap aggregation with cross-gap averaging
- Diameter categories: <=8, <=28, <=48, <=68, <=88, >88 cm

## CuPy JIT Limitations

SAGESim uses CuPy's `jit.rawkernel` for GPU compilation. This imposes strict constraints on step function code:

**Unsupported Python Features:**
- Dicts and objects
- `*args` and `**kwargs`
- Nested functions
- `return` statements (unreliable)
- `break` and `continue` statements
- For-each loops (use `for i in range(n)` only)

**Critical Gotchas:**
- NaN checks must use inequality: `not cp.isnan(x)` or `x != x`
- Use CuPy datatypes and functions, not NumPy
- `-1` indexing accesses last memory block element, not logical array element - use `len(array)-1` instead
- Cannot reassign variables within `if` or `for` blocks - declare at function top level

**Data Access Patterns:**
- Read from original tensors: `get_this_agent_data_from_tensor(agent_index, tensor)`
- Write to write buffers: `set_this_agent_data_from_tensor(agent_index, write_tensor, value)`
- Access neighbors: `get_neighbor_data_from_tensor(agent_ids, neighbor_id, tensor)`

## NLCD Forest Data

The project includes utilities for downloading National Land Cover Database (NLCD) forest data for the Southeast US region.

### Manual Download (Recommended)

Automated downloads are restricted. Follow instructions in `NLCD_DOWNLOAD_INSTRUCTIONS.md`:

1. Visit https://www.mrlc.gov/viewer/
2. Download NLCD 2021 Land Cover for Southeast US (West: -94°, East: -75°, North: 39.5°, South: 24.5°)
3. Place .tif file in `nlcd_data/`

### Plotting

```bash
uv run python plot_southeast_forests.py
```

Creates visualizations of forest distributions. Works with downloaded NLCD data or generates simulated data for testing.

## Project Structure Notes

- `SAGESim/` is a git submodule - contains the full SAGESim framework and examples
- `GAPpy/` is a git submodule - Python translation of UVAFME (reference implementation)
- `input_data/` contains UVAFME CSV input files (species list, site data, climate, climate stddev, range list, altitudes)
- `output_data/` stores simulation CSV outputs
- `gap/` is the main GGap implementation
- `gap/step_functions/` contains GPU step functions organized by agent type (tree/, gap/, site/)
- `gap/docs/` contains detailed documentation (AGENT_PROPERTIES.md, implementation_logic.md)

## Development Guidelines

### Adding New Tree Processes

When implementing UVAFME processes as GPU kernels:

1. Study the original UVAFME Fortran code and Python translation
2. Identify state dependencies and neighbor interactions
3. Design step function respecting CuPy JIT constraints
4. Use SAGESim's double-buffering pattern for any writes
5. Register properties and step functions in the breed class
6. Test with small agent counts before scaling up

### MPI Parallelization

- Each MPI rank processes a partition of agents
- Use `sync_workers_every_n_ticks` to control synchronization frequency
- Global properties are reduced across ranks using `reduce_global_data_vector`
- Agents can access neighbor data across rank boundaries via contextualization

### Debugging GPU Kernels

- Start with CPU mode: `model.setup(use_gpu=False)` (if supported)
- Print statements don't work in GPU kernels - use global properties for debugging
- Check generated kernel code in `step_func_code.py`
- Use small agent counts and few ticks for rapid iteration

---
> Source: [bioatmosphere/GGap](https://github.com/bioatmosphere/GGap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
