## simulation-evaluation

> Containerized Python tool for evaluating hydrological simulation performance against observed data using standard metrics (NSE, KGE, R², MSE, RMSE).

# Simulation Evaluation Tool

## Purpose
Containerized Python tool for evaluating hydrological simulation performance against observed data using standard metrics (NSE, KGE, R², MSE, RMSE).

## Key Components
- **Python Core** (`src/`): Data loading, metric calculation, output generation
- **Svelte Report App** (`src/report/`): Interactive HTML reports with visualizations  
- **Docker Setup**: Containerized execution following tool-spec standard
- **Input Data**: CAMELS-DE hydrological catchment data (CSV/Parquet)

## Workflow
1. Load simulation/observation data from `/in/` using wildcard patterns
2. Calculate performance metrics for each catchment
3. Generate CSV/JSON metrics summary
4. Build interactive HTML report with time series plots

## Technologies
- **Backend**: Python (pandas, numpy, scipy, sklearn)
- **Frontend**: SvelteKit, TypeScript, Plotly.js, Tailwind CSS
- **Containerization**: Docker with tool-spec compliance
- **Data Processing**: Supports CSV/Parquet, flexible column mapping

## Key Files
- `run.py`: Main entrypoint and orchestration
- `evaluation.py`: Core metric calculations (NSE, KGE, etc.)
- `outputs.py`: Report generation and data compression
- `tool.yml`: Tool specification and parameter definitions
- `src/report/`: Svelte application for interactive visualization

## Development Notes
- Uses json2args for parameter parsing from `/in/input.json`
- Supports both separate and combined observation/simulation files
- Generates compressed datasets for web visualization
- Follows tool-spec container structure (`/in`, `/out`, `/src`)

---
> Source: [VForWaTer/simulation_evaluation](https://github.com/VForWaTer/simulation_evaluation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
