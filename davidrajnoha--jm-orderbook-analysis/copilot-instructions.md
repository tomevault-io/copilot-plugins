## jm-orderbook-analysis

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a research project for analyzing Joinmarket orderbook snapshots as part of a master thesis. The codebase parses JSON orderbook data and creates visualizations to answer questions about liquidity, maker offer lifetimes, and coinjoin protocol dynamics.

## Development Environment

**Package Management**: Poetry
- Use `poetry install` to set up dependencies
- Use `poetry shell` to activate the virtual environment
- Dependencies managed in `pyproject.toml`

**Testing**: pytest
- Run tests with `poetry run pytest`
- Test files located in `test/` directory
- Main test modules: `test_snapshot_processing.py`, `test_extended_snapshot_data.py`, `test_visualisations.py`

**Data Analysis Environment**: Jupyter notebooks
- Start with `poetry run jupyter notebook`
- Analysis notebooks located in `src/` directory

## Architecture

The codebase follows a modular structure with three main components:

### Data Processing Pipeline (`src/preprocessing/`)
- **`snapshot.py`**: Core data loading and orderbook snapshot processing
  - `load_and_process_snapshot()`: Main entry point for processing individual snapshots
  - `process_offers()`: Analyzes offer data to extract liquidity, fees, and order sizes
  - `compute_statistics()`: Calculates statistical measures from processed offers
- **`dataframe.py`**: Batch processing and DataFrame management
  - `load_snapshots_to_dataframe()`: Processes multiple snapshots with timestamp filtering
  - Handles data serialization with pickle format for performance
- **`utils.py`**: Utility functions for file processing and timestamp extraction
- **`maker_offers.py`**: Specialized processing for maker-specific analysis

### Analysis Modules (`src/analysis/`)
- **`offer_changes.py`**: Advanced streaming analysis for detecting offer modifications
  - `StreamingOfferAnalyzer`: Single-pass algorithm for tracking maker behavior changes
  - `OfferSignature`: Data structure for comparing similar offers
  - Real-time change detection (appeared, disappeared, modified, quantity changes)
- **`fees.py`**: Fee analysis and calculations

### Visualization (`src/visualisations/`)
- **`plot.py`**: Core plotting utilities
- **`fees.py`**: Fee-specific visualizations

## Data Structure

**Raw Data**: JSON files containing orderbook snapshots
- Located in timestamped directories (e.g., `2025-jsons/2025-07-27/`)
- Format: `orderbook_HH-MM.json`
- Contains `offers` and `fidelitybonds` arrays

**Processed Data**: 
- Cached as pickle files (`dataframe.pkl`) for performance
- Structured DataFrames with temporal analysis capabilities

## Key Processing Concepts

**Offer Types**:
- `sw0reloffer`: Relative fee offers (percentage-based)
- `sw0absoffer`: Absolute fee offers (fixed satoshi amounts)

**Timestamp Filtering**: Built-in capability to process data with minimum time intervals to manage large datasets efficiently

**Streaming Analysis**: Single-pass algorithms for real-time change detection without loading entire datasets into memory

## Common Workflows

**Loading and Processing Data**:

```python
from src.src.preprocessing.dataframe import load_snapshots_to_dataframe

df = load_snapshots_to_dataframe(filepaths, min_interval_minutes=60)
```

**Analyzing Offer Changes**:

```python
from src.src.analysis import StreamingOfferAnalyzer

analyzer = StreamingOfferAnalyzer()
summary = analyzer.analyze_complete_dataset(df_offers)
```

**Data Persistence**:

```python
from src.src.preprocessing.dataframe import save_dataframe, load_dataframe

save_dataframe(df, 'processed_data.pkl')
df = load_dataframe('processed_data.pkl')
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DavidRajnoha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DavidRajnoha)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
