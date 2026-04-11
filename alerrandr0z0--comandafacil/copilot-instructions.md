## comandafacil

> This is a Python-based machine learning project designed to estimate the number of beach kiosks across Brazilian coastal municipalities. It utilizes demographic, economic, and geographic data (such as population, GDP per capita, occasional households, and coastal length) to power statistical and machine learning models, specifically a Poisson regression model and a Random Forest Regressor.

# Project Overview

This is a Python-based machine learning project designed to estimate the number of beach kiosks across Brazilian coastal municipalities. It utilizes demographic, economic, and geographic data (such as population, GDP per capita, occasional households, and coastal length) to power statistical and machine learning models, specifically a Poisson regression model and a Random Forest Regressor.

The core logic resides in the `src/` directory, while datasets and trained models are stored in `data/` and `models/` respectively. The project acts as a pipeline that handles offline database setup, geographic metadata extraction, model training, and unified inference via a CLI.

**Main Technologies:**
- Python 3
- Data manipulation: `pandas`, `numpy`
- Machine Learning / Stats: `scikit-learn`, `statsmodels`
- Geospatial: `geopandas`, `shapely`, `pyproj`
- Plotting: `matplotlib`, `seaborn`
- API interaction: `requests` (for IBGE and OSM data)

# Building and Running

The project does not use a standard build system (like `setuptools` or `poetry`). Instead, it relies on a local virtual environment and a sequence of execution scripts.

### Environment Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Data Preparation and Training Pipeline

Since there is no standard build step, building the project implies preparing the datasets and training the models. Execute these commands from the repository root:

1. **Build the local offline database (IBGE-derived):**
   ```bash
   PYTHONPATH=$PYTHONPATH:. ./.venv/bin/python src/setup_offline_db.py
   ```
2. **Build geographic coastal metadata:**
   ```bash
   PYTHONPATH=$PYTHONPATH:. ./.venv/bin/python src/build_geo_metadata.py
   ```
3. **Train the models:**
   ```bash
   PYTHONPATH=$PYTHONPATH:. ./.venv/bin/python src/train.py
   ```

### Running the CLI

The main entry point for predictions is the unified CLI `src/run.py`:

- **Municipal scope (single city by IBGE ID):**
  ```bash
  PYTHONPATH=$PYTHONPATH:. ./.venv/bin/python src/run.py --scope municipal --city-id 2411056
  ```
- **State scope (e.g., RN):**
  ```bash
  PYTHONPATH=$PYTHONPATH:. ./.venv/bin/python src/run.py --scope estadual --uf RN --prefer-local-cache
  ```
- **National scope:**
  ```bash
  PYTHONPATH=$PYTHONPATH:. ./.venv/bin/python src/run.py --scope nacional --prefer-local-cache
  ```

### Adding Calibration Data

You can add manual calibration points and retrain the models dynamically:
```bash
PYTHONPATH=$PYTHONPATH:. ./.venv/bin/python src/online_trainer.py <IBGE_CITY_ID> <REAL_KIOSK_COUNT>
```

### Testing

There is no formal testing suite (like `pytest`) implemented yet. To validate your changes, rely on:
1. Syntax validation: `source .venv/bin/activate && python3 -m compileall src`
2. Import smoke test: `source .venv/bin/activate && PYTHONPATH=$PYTHONPATH:. python3 -c "from src.model_engine import KioskModelEngine; from src.ibge_client import IBGEClient"`
3. Running a prediction test with existing models.

# Development Conventions

- **Formatting & Style:** 
  - Follow PEP 8 guidelines.
  - Use 4-space indentation (no tabs).
  - Target a maximum line length of 88-100 characters.
  - `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants.
  - Retain Portuguese names for domain-specific dataset fields (e.g., `municipio`, `codigo_ibge`) for continuity.
- **Type Hinting:** Existing code is mostly untyped, but type hints (`pd.DataFrame`, `dict`, etc.) are encouraged for new code, particularly for public methods and CLI entry points.
- **Error Handling:** Avoid bare `except:` statements. Use explicit exceptions (e.g., `except Exception as e:`) and prefer fail-soft behavior (log and continue) for batch data processing loops across cities to prevent breaking the entire pipeline.
- **Data Safeties:** 
  - Ensure defensive parsing for IBGE responses due to potentially inconsistent schemas.
  - Maintain safety floor variables in preprocessing (e.g., `cnae_adjusted`, `coast_km`) so models don't learn 0 as a strict economic fact.
  - Always validate input rows before training/prediction (e.g., `populacao > 0`, `pib_bilhoes > 0`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alerrandr0z0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Alerrandr0z0)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
