## psilocybin-eeg

> This project analyzes EEG neural data from participants recorded during listening to two types of music (classical and psytrance) under two conditions (placebo and psilocybin treatment). The major goal is to find and analyze the **inter-subject synchrony (ISC)** between participants.

# Psilocybin-EEG Project - Agent Instructions

## Project Context

This project analyzes EEG neural data from participants recorded during listening to two types of music (classical and psytrance) under two conditions (placebo and psilocybin treatment). The major goal is to find and analyze the **inter-subject synchrony (ISC)** between participants.

### Experimental Design
- **EEG System**: EGI GSN-HydroCel 257-channel montage
- **Conditions**: Placebo / Psilocybin (within-subject, counterbalanced A/B)
- **Music Types**: Classical (`CLASSIC`), Psytrance (`PSYTRANCE`)
- **Primary Analysis**: Inter-subject correlation (ISC) per frequency band
- **File Formats**: EDF (raw), MNE `.fif` (processed), NumPy `.npy` (arrays), CSV (metadata)

## Technology Stack
- **Python**: ≥3.12
- **Core Libraries**: MNE-Python (EEG processing), NumPy, SciPy, Pandas
- **ICA/Artifact Removal**: AutoReject, MNE-ICALabel, PyTorch (optional GPU)
- **Visualization**: Matplotlib, Seaborn
- **Testing**: pytest
- **Linting**: Ruff (via GitHub Actions)

## Pipeline Architecture

The analysis pipeline has three main stages:

### 1. Preprocessing (`src/preprocessing/`, `scripts/run_preprocessing.py`)
The pipeline order is: **channel_prep → filtering → ica → time_alignment**

For the full step-by-step details of each preprocessing stage, see [`docs/preprocessing_steps.md`](../docs/preprocessing_steps.md).

### 2. Time Alignment (`src/preprocessing/time_alignment.py`, `scripts/run_time_alignment.py`)
- Extract TAG channel stimulus markers
- Compute pairwise cross-correlation
- Select reference recording
- Shift and crop all recordings to a common time window

### 3. Analysis (`src/analysis/`, `scripts/run_analysis.py`)

The analysis workflow follows three principles:
1. **Exploration first** — each analysis must be sketched in a Jupyter notebook before production implementation.
2. **Statistical plots in Seaborn** — use Seaborn for all statistical visualizations where possible; fall back to Matplotlib only for EEG topomaps and other domain-specific plots.
3. **Reproducible production pipeline** — every analysis must have a structured CLI script (`scripts/`) and a corresponding HPC job template (`jobs/metacentrum/`) so it can be run on the cluster for large-scale processing.

### Notebook Organisation

All exploration notebooks live under `notebooks/` in numbered subdirectories:

```
notebooks/
├── 00-preprocessing/                # Preprocessing inspection and time alignment
│   ├── eeg_data_inspection.ipynb   # Inspect raw EEG at different pipeline stages
│   └── time_alignment.ipynb        # Time alignment, cross-correlation, TAG overlay
├── 01-raw-mean-variance-analysis/   # Mean-variance synchrony analysis
│   ├── mean_variance_broadband.ipynb   # Part 1 — broadband (z-scored raw)
│   └── mean_variance_bands.ipynb       # Part 2 — per-frequency-band
├── 02-<analysis-name>/              # Next analysis (use next available 2-digit ID)
│   └── ...
└── ...                              # Legacy / unorganised notebooks (do not move)
```

**Naming rules:**
- Subdirectory: `NN-<kebab-case-analysis-name>` where `NN` is a two-digit zero-padded integer. Use `00` for preprocessing notebooks (e.g. `00-preprocessing`) and start analysis notebooks at `01` (e.g. `01-raw-mean-variance-analysis`, `02-isc-analysis`).
- Notebook files: descriptive `snake_case` names that reflect the scope (e.g. `mean_variance_broadband.ipynb`, `mean_variance_bands.ipynb`).
- When a single analysis covers multiple distinct sub-scopes (e.g. broadband vs per-band), **split into separate notebooks** — one notebook per logical sub-scope.

**Notebook structure (each notebook must follow this template):**
1. **Setup cell** (code) — **must be the very first cell**. Contains the project-root resolver (`_p`) and **all** imports from `sys`, `os`, `pathlib`, third-party libraries (`numpy`, `matplotlib`, `seaborn`, `pandas`), `src.*`, and `scripts.*`. Imports that follow non-import code (e.g. the `sys.path.insert` resolver loop) must carry `# noqa: E402` so the CI linter passes. Only import what is needed for this notebook's scope. **Never import inside analysis cells.**
2. **Title cell** (Markdown) — analysis name + scope + brief description of what is computed and visualised.
3. **Configuration cell** (code) — all user-tunable parameters (condition, music types, window sizes, thresholds …) in one place; also define `SAVE_PLOTS = True` and `PLOTS_DIR = ProjectPaths.NOTEBOOKS_DIR / "<notebook-dir>" / "plots" / "<scope>"` (import `ProjectPaths` in the setup cell).
4. **Data loading cell** (code) — load / process-and-save data via `load_analyzers` / `analyzers_to_datasets`.
5. **Dataset selection cell** (code) — pick the active music-type label and look up the dataset using the combined key `f"{CONDITION.value}_{LABEL}"` (e.g. `datasets[f"{CONDITION.value}_{LABEL}"]`). `load_analyzers` / `analyzers_to_datasets` key datasets as `"{condition}_{music_type}"` (e.g. `"Placebo_CLASSIC"`). Derive dimension variables (`n_subjects`, `n_channels`, `n_times`, `sfreq`) from the selected `AnalysisData` object.
6. **One cell per analysis step** — each step has a Markdown header explaining what is computed/plotted, followed by a single code cell that calls one `src.analysis.*` or `src.visualization.*` function and displays the result; pass `save_path=PLOTS_DIR / "filename.png" if SAVE_PLOTS else None` to each plot function.

**Notebook code style:**
- The setup cell must be the **first cell** in the notebook (cell index 0). The title/description markdown can come after it.
- All imports must be at the top level in the setup cell — never scatter imports across analysis cells.
- Imports that follow a `sys.path.insert` block in the setup cell must have `# noqa: E402` to satisfy Ruff's E402 rule.
- Follow **Ruff** linting rules and **Black** formatting conventions in all notebook code cells (same as production code): use double quotes for strings, add trailing commas in multi-line collections, keep lines ≤ 88 characters, and avoid unused imports or variables.

> When developing a new analysis, always create a new numbered subdirectory and at least one notebook following the structure above **before** writing production code in `src/`.

### Plot Output Directories

Plots are saved in two different locations depending on the context:

- **Jupyter notebooks** — save figures to a `plots/` subdirectory *inside* the notebook's own directory. For example, notebooks under `notebooks/01-raw-mean-variance-analysis/` should save to `notebooks/01-raw-mean-variance-analysis/plots/broadband/<analysis_type>/` (broadband) or `notebooks/01-raw-mean-variance-analysis/plots/bands/<analysis_type>/` (per-band).
- **CLI scripts** — save figures to `plots/<notebook-directory-name>/<Condition>_<MusicType>/broadband/<analysis_type>/` (broadband) or `plots/<notebook-directory-name>/<Condition>_<MusicType>/bands/<analysis_type>/` (per-band).

### Canonical plot directory structure

The canonical structure for all CLI-script output (used by the Results Browser) is:

```
plots/
└── {NN}-{analysis-name}/            ← analysis stage (matches notebook subdirectory)
    └── {Condition}_{MusicType}/     ← condition + music type (e.g. Placebo_CLASSIC)
        ├── broadband/               ← broadband (no per-band filtering)
        │   └── {analysis_type}/     ← specific analysis (e.g. loo_isc, timeseries)
        │       └── *.png
        └── bands/                   ← per-frequency-band analyses
            └── {analysis_type}/     ← specific analysis (e.g. loo_isc, windowed)
                └── *.png            ← filename includes band name for per-band files
```

**Rules:**
1. **Never** use `raw/` — broadband analyses always go into `broadband/<analysis_type>/`.
2. **Never** place files directly in `broadband/` or `bands/` — always use an `<analysis_type>/` subdirectory.
3. `<analysis_type>` names must be consistent across `broadband/` and `bands/` where the same analysis is run for both (e.g. both use `loo_isc`, `pairwise_isc`, `sliding_window`).
4. Mean-field and other broadband-only analyses go in `broadband/mean_field/` (or appropriate name).
5. Per-band filenames should include the band name as the first token (e.g. `delta_loo_isc_distribution.png`).

## Directory Structure

```
psilocybin-eeg/
├── config/                        # Electrode coordinates, excluded electrodes, participant maps
├── data/                          # Raw, interim, and processed data (gitignored)
│   ├── raw/psilo_music/           # Original EDF files
│   ├── interim/psilo_music/       # Before/after ICA, IC probabilities
│   └── processed/psilo_music/     # Cleaned EEG, cropped, concatenated
├── src/                           # Source package
│   ├── definitions/               # Enums, constants, field definitions
│   ├── io/                        # Parsing, loading, saving
│   ├── preprocessing/             # Channel prep, filtering, ICA, alignment
│   ├── analysis/                  # ISC, wavelet, mean/variance, summary
│   ├── visualization/             # Plots for preprocessing and analysis
│   ├── filtering/                 # Metadata filtering utilities
│   └── utils/                     # Logging helpers
├── scripts/                       # CLI entry points
├── notebooks/                     # Jupyter notebooks for exploration (numbered subdirs: NN-<name>/)
│   │                              #   00-preprocessing/ — preprocessing inspection and time alignment
│   │                              #   01-<analysis>/ and higher — analysis notebooks
├── tests/                         # pytest test suite
├── jobs/                          # HPC job scripts (Metacentrum)
└── docs/                          # Extended documentation
```

## Coding Standards

### Python Style
- Follow **PEP 8** and **Ruff linter** rules (enforced via GitHub Actions)
- Use **type hints** for all function parameters and return values
- Function/variable/file names: `snake_case`
- Class names: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Private attributes/methods: prefix with `_`

### Naming Conventions
- **DataFrame columns**: `snake_case` (e.g., `"condition"`, `"music_type"`, `"participant_id"`)
- **Enum values**: Use enum classes from `src/definitions/fields.py`:
  - `ConditionVariants.PLACEBO`, `ConditionVariants.PSILOCYBIN`
  - `MusicTypeVariants.CLASSICAL`, `MusicTypeVariants.PSYTRANCE`
  - `FrequencyBandNames.DELTA`, `FrequencyBandNames.THETA`, etc.
- **File paths**: Use `ProjectPaths` constants from `src/definitions/constants.py`
- **Never hardcode categorical values as strings** — always use Enums

### Documentation
- **Docstrings** for all public functions and classes (Google style)
- Explain *why* and *what is non-obvious* — do not restate what is already clear from the function signature or type hints
- Add comments for complex logic or non-obvious decisions
- Keep comments concise and up-to-date with code changes

### Error Handling
- Use descriptive error messages
- Validate input data dimensions and types early
- Log errors using `src/utils/logging_config.py` utilities
- Prefer explicit validation over silent failures

## Module-Specific Guidelines

### `src/definitions/`
- **Single source of truth** for all categorical values (Enums)
- Never hardcode strings; always reference Enum values
- All field names defined in `fields.py`
- Path constants in `constants.py` (use `ProjectPaths`)
- Channel mappings in `mappings.py`

### `src/io/`
- **Parsing** (`parsing.py`): Extract metadata from filenames (condition, music type, participant)
- **Loading** (`loading.py`): Load EEG data, route by data type, construct paths
- **Saving** (`saving.py`): Save data with proper naming conventions
- Always preserve metadata through load/save operations

### `src/preprocessing/`
- Follow the pipeline order: **channel_prep → filtering → ica → time_alignment**
- Use MNE-Python conventions for `Raw` and `Epochs` objects
- Always preserve channel information and metadata
- Document all preprocessing parameters (filter cutoffs, ICA thresholds, etc.)
- Use `LoggerMixin` from `src/utils/logging_config.py` for class logging

### `src/analysis/`
- **ISC functions** (`isc.py`): Validate input shape `(n_subjects, n_channels, n_timepoints)` before computation
- Return results as **pandas DataFrames** with meaningful indices (participant, condition, music, band, channel)
- Include metadata columns for traceability
- Document all statistical assumptions and methods
- **Data representations** (`data_representations.py`): Use `AnalysisData` classes to encapsulate data + metadata

### `src/visualization/`
- Use consistent color schemes (defined in module-level constants)
- All plots should have clear titles, labels, and legends
- Save plots with descriptive filenames including metadata (condition, music, participant)
- Use **Seaborn** for statistical plots, Matplotlib for EEG topomaps

### `src/filtering/`
- Use `DatasetFilter` class for metadata-based filtering
- Support filtering by condition, music type, participant, frequency band
- Return filtered DataFrames with preserved indices

### `tests/`
- **Coverage target**: 90%+ for all modules
- Test file structure: `tests/test_<module>.py` for each `src/<package>/<module>.py`
- Use pytest fixtures from `conftest.py` for common test data
- All tests must pass before PR merge (`python -m pytest tests/ -v`)
- Mock file I/O operations where appropriate
- Test edge cases (empty data, missing values, invalid inputs)

## Key Enums and Constants

### From `src/definitions/fields.py`
- **Conditions**: `ConditionVariants.PLACEBO`, `ConditionVariants.PSILOCYBIN`
- **Music Types**: `MusicTypeVariants.CLASSICAL`, `MusicTypeVariants.PSYTRANCE`
- **Frequency Bands**: `FrequencyBandNames.DELTA`, `.THETA`, `.ALPHA`, `.BETA`, `.GAMMA`
- **Analysis Types**: `AnalysisVariants.ISC`, `.MEAN_VARIANCE`, `.WAVELET_POWER`, `.WAVELET_PHASE`
- **Data Types**: `PreprocessedDataVariants.RAW_BEFORE_ICA`, `.RAW_AFTER_ICA`, `.RAW_CROPPED`, `.CONCATENATED`, `.IC_PROBABILITIES`
- **Coordinate Systems**: `CoordinateSystems.HYDROGEL_257`, `.HYDROGEL_257_NO_FIDUCIALS`

### Frequency Band Definitions
```python
from src.definitions.fields import FrequencyBandNames

FREQUENCY_BANDS = {
    FrequencyBandNames.DELTA.value: (1, 4),
    FrequencyBandNames.THETA.value: (4, 8),
    FrequencyBandNames.ALPHA.value: (8, 13),
    FrequencyBandNames.BETA.value: (13, 30),
    FrequencyBandNames.GAMMA.value: (30, 70),
}
```

## Common Workflows

### Adding a New Analysis
1. **Create a numbered notebook subdirectory** — `notebooks/NN-<kebab-case-name>/` using the next available two-digit ID (e.g. `02-isc-analysis`).
2. **Sketch in a Jupyter notebook** — follow the 6-cell template (title → setup → config → data loading → dataset selection → one cell per step); split into multiple notebooks if the analysis covers distinct sub-scopes.
3. Create the implementation in the appropriate `src/analysis/` module
4. Add comprehensive docstring with input/output specs; explain non-obvious logic
5. Use type hints for all parameters and return values
6. Validate input data dimensions and types
7. Return pandas DataFrame with proper metadata columns
8. Create a CLI script in `scripts/` that exposes the analysis
9. Add an HPC job template in `jobs/metacentrum/NN-<kebab-case-analysis-name>/` (e.g. `jobs/metacentrum/01-raw-mean-variance-analysis/run_mean_variance.pbs`) matching the same numbered directory name used for notebooks
10. Write tests in `tests/test_<module>.py`
11. Run tests: `python -m pytest tests/test_<module>.py -v`

### Modifying Preprocessing Steps
1. Edit the appropriate `src/preprocessing/` module
2. Ensure changes preserve metadata and channel information
3. Document parameter changes in docstrings
4. Update `docs/preprocessing_steps.md` if pipeline stages change
5. Run full preprocessing test suite: `python -m pytest tests/test_channel_prep.py tests/test_filtering_preprocessing.py tests/test_ica.py -v`
6. Verify no regressions in downstream analysis

### Working with Data
1. **Never hardcode data paths** — use `ProjectPaths` from `src/definitions/constants.py`
2. Use `src/io/parsing.py` functions to extract metadata from filenames
3. Route all data loading through `src/io/loading.py`
4. Route all data saving through `src/io/saving.py`
5. Follow naming conventions from `data/README.md`
6. Add new participants to exclusion list in `data/excluded_participants/psilo_music.csv` if needed

### Running Scripts
```bash
# Preprocessing
python scripts/run_preprocessing.py --raw_processing --plot_results

# Time alignment
python scripts/run_time_alignment.py --condition Placebo --music_type CLASSIC

# ISC analysis (dedicated script — Placebo condition by default)
python scripts/run_isc.py --music_type CLASSIC PSYTRANCE

# ISC with custom multi-scale window sizes and channel subsampling
python scripts/run_isc.py --music_type CLASSIC \
    --window_fine_sec 1.0 --window_sec 5.0 --window_large_sec 15.0 \
    --n_ch_subsample 64

# Mean-variance analysis (Placebo condition by default)
python scripts/run_mean_variance.py --music_type CLASSIC PSYTRANCE

# Wavelet analysis (Placebo condition by default)
python scripts/run_analysis.py --analysis wavelet_power --wavelet_cache_dir data/processed/psilo_music/wavelets
```

## Dos and Don'ts

### Do
✓ Use Enum types for all categorical values (condition, music type, frequency bands)
✓ Add type hints to all functions
✓ Write tests for new functionality (aim for 90%+ coverage)
✓ Follow existing project structure and naming conventions
✓ Use logging utilities from `src/utils/logging_config.py`
✓ Preserve metadata through all pipeline stages
✓ Validate input data dimensions early
✓ Return pandas DataFrames from analysis functions
✓ Sketch new analyses in a Jupyter notebook before implementing them (use the numbered subdirectory structure in `notebooks/`)
✓ Run linter and tests before committing (`ruff check .`, `pytest tests/ -v`)
✓ **Keep documentation in sync with code** — update `src/analysis/README.md`, `src/preprocessing/README.md`, `docs/preprocessing_steps.md`, and this file whenever you add, rename, or remove modules, functions, classes, or CLI flags
✓ **Default to Placebo condition** — all new analyses, notebooks, CLI scripts, and HPC jobs must use `ConditionVariants.PLACEBO` as the default/only condition unless explicitly instructed otherwise

### Don't
✗ Hardcode categorical values as strings (use Enums instead)
✗ Skip type hints on new functions
✗ Modify the preprocessing pipeline order without documentation
✗ Commit data files to git (they're gitignored)
✗ Remove or disable existing tests
✗ Add dependencies without updating `requirements.txt` and `pyproject.toml`
✗ Use deprecated MNE-Python functions (check MNE docs)
✗ Create circular imports between modules
✗ Mutate input data without copying first
✗ Use global state or class-level mutable defaults
✗ Leave documentation stale — if you find an error or omission in any README or doc file, correct it in the same commit
✗ **Run analysis on Psilocybin data** unless explicitly instructed — all analyses default to Placebo only

## Testing Requirements

### Before Committing
```bash
# Run full test suite
python -m pytest tests/ -v

# Run tests for specific module
python -m pytest tests/test_isc.py -v

# Check code style
ruff check .

# Auto-fix formatting
ruff format .
```

### Test Coverage
- All new functions must have corresponding tests
- Test both happy path and edge cases
- Use parametrized tests for multiple input variations
- Mock file I/O to avoid dependencies on actual data files

## HPC Execution

For large-scale processing on the Metacentrum HPC cluster:
- Job scripts in `jobs/metacentrum/NN-<kebab-case-analysis-name>/` — use the **same numbered subdirectory name** as the corresponding notebooks directory (e.g. `jobs/metacentrum/01-raw-mean-variance-analysis/run_mean_variance.pbs`)
- Preprocessing job scripts live in `jobs/metacentrum/00-preprocessing/` (matching the `notebooks/00-preprocessing/` convention)
- Legacy job scripts that predate this convention live directly in `jobs/metacentrum/` and should not be moved
- Request appropriate resources (CPU, memory, GPU for ICA)
- Use `--verbose` flags for detailed logging in HPC jobs
- See `docs/hpc_guide.md` for resource requirements

## Documentation Resources

- [`CODEBASE_STRUCTURE.md`](../CODEBASE_STRUCTURE.md) — Full annotated directory tree
- [`docs/pipeline_overview.md`](../docs/pipeline_overview.md) — Detailed pipeline stages
- [`docs/preprocessing_steps.md`](../docs/preprocessing_steps.md) — Step-by-step preprocessing details
- [`docs/data_dictionary.md`](../docs/data_dictionary.md) — All field names and enum values
- [`docs/hpc_guide.md`](../docs/hpc_guide.md) — HPC job submission guide
- [`data/README.md`](../data/README.md) — Data directory layout and naming
- [`src/preprocessing/README.md`](../src/preprocessing/README.md) — Preprocessing details
- [`src/analysis/README.md`](../src/analysis/README.md) — Analysis modules, data shape conventions, and ISC workflow

## Visualization Catalog (`viz_catalog/`)

A self-contained **Streamlit multi-page app** that serves as an interactive analysis reference and a local results file browser. It lives entirely under `viz_catalog/` and has **no imports from `src/`** — it is independent of the main project package.

### Structure

```
viz_catalog/
├── app.py                        ← Streamlit entry point (page config + home)
├── catalog.yaml                  ← Single source of truth for all analyses/plots
├── requirements.txt              ← streamlit, pyyaml, matplotlib, numpy
├── README.md                     ← User-facing run instructions
└── pages/
    ├── 1_📋_Catalog.py           ← Analysis catalog (YAML-driven, sketch functions)
    └── 2_🔬_Results_Browser.py   ← Local plot-file browser
```

### Running locally

```bash
# With uv (recommended) — one-liner, no prior install:
uv run --with "streamlit>=1.32.0" --with "pyyaml>=6.0" --with "numpy>=1.24.0" --with "matplotlib>=3.7.0" \
    streamlit run viz_catalog/app.py

# With uv — persistent venv:
uv venv .venv-viz && source .venv-viz/bin/activate
uv pip install -r viz_catalog/requirements.txt
streamlit run viz_catalog/app.py

# With plain pip:
pip install -r viz_catalog/requirements.txt
streamlit run viz_catalog/app.py
```

The app opens at **`http://localhost:8501`**.

### Adding a new analysis group (YAML only — no Python needed)

Add a new entry under `analyses:` in `viz_catalog/catalog.yaml`:

```yaml
analyses:
  - id: "03-wavelet"
    title: "Stage 03 — Wavelet Analysis"
    description: "Time-frequency decomposition using Morlet wavelets."
    notebooks:
      - path: "notebooks/03-wavelet-analysis/wavelet_power_exploration.ipynb"
        label: "Wavelet Power Exploration"
    plots:
      - id: "tf_map"
        title: "Time-frequency map"
        notebook: "notebooks/03-wavelet-analysis/wavelet_power_exploration.ipynb"
        section: "TF decomposition"
        filename_pattern: "tf_map*.png"
        data_shape:
          input: "(n_channels, n_times)"
          output: "(n_frequencies, n_times) — power per freq × time"
        operation_order:
          - "Apply complex Morlet wavelet"
          - "Compute instantaneous power"
          - "Average across channels"
        interpretation: "Bright regions = high power at that frequency and time."
        sketch_type: "timeseries_heatmap"   # must match a key in SKETCH_FUNCTIONS
```

Every plot entry **must** include: `id`, `title`, `notebook`, `section`, `filename_pattern`, `data_shape` (with `input` and `output`), `operation_order` (list), `interpretation`, and `sketch_type`.

### Adding a new sketch type (Python)

1. Open `viz_catalog/pages/1_📋_Catalog.py`.
2. Write a zero-argument function that returns a `matplotlib.figure.Figure`. Keep it small (`figsize=(5, 2.5)`) and schematic — it is an orientation aid, not a data plot.
3. Register it in the `SKETCH_FUNCTIONS` dict at module level.

```python
def sketch_my_new_type() -> Figure:
    fig, ax = plt.subplots(figsize=(5, 2.5))
    # ... draw sketch ...
    ax.set_title("My New Type", fontsize=8)
    fig.tight_layout()
    return fig

SKETCH_FUNCTIONS["my_new_type"] = sketch_my_new_type
```

4. Use `sketch_type: "my_new_type"` in `catalog.yaml`.

**Existing sketch types** (16 registered): `timeseries_multichannel`, `psd`, `topomap`, `timeseries_offset`, `matrix_heatmap`, `timeseries_variance`, `histogram`, `timeseries_multisubject`, `timeseries_multiband`, `histogram_overlay`, `timeseries_heatmap`, `bar_grouped`, `histogram_facet`, `bar_grouped_bands`, `grid_timeseries_heatmap`, `raster_overlap`.

### Catalog page conventions (`1_📋_Catalog.py`)

- Load `catalog.yaml` via `@st.cache_data` — **never** re-parse the file on every render.
- Render analyses in a **2-column grid** using `st.columns(2)`.
- Each plot card must show: title, data shape table (input → output), numbered operation order, `st.info` interpretation box, and the matplotlib sketch via `st.pyplot`.
- GitHub notebook links must point to the **`develop` branch** (`GITHUB_BASE = "https://github.com/dbeinhauer/psilocybin-eeg/blob/develop"`). Update this constant if the default branch changes.
- Sketch functions must use `numpy.random.default_rng(<seed>)` (not `np.random.seed`) so they are reproducible and side-effect-free.
- Wrap each sketch in `try/except` and show `st.warning` if the sketch type is unknown — never crash the page.

### Results Browser page conventions (`2_🔬_Results_Browser.py`)

- Accept a free-text directory path from the user; use `pathlib.Path.rglob("*.png")` + `rglob("*.jpg")` to build the file index.
- Only index files matching the canonical plot structure: `<stage>/<Condition>_<MusicType>/<broadband|bands>/<analysis_type>/<filename.ext>` (5 path components minimum). Skip everything else silently.
- Parse path tokens from the relative path, not from filenames: `stage` → parts[0], `condition_music` → parts[1], `spectrum_type` → parts[2] (`broadband` or `bands`), `analysis_type` → parts[3].
- Sidebar filters (all default to empty — no images shown until at least one is selected): analysis stage, condition, music type, spectrum type, analysis type, frequency band, free-text filename search.
- Frequency band: for `broadband/` files → band = `"broadband"`; for `bands/` files → extract whole-token band name from filename stem.
- Support **grid view** (configurable columns slider) and **list view** (filename + size + path metadata).
- **Compare mode**: 3-way radio — None / Manual (select 2–4 side-by-side) / By condition/music type (auto-find all variants of a filename stem).
- Cache the file scan with `@st.cache_data(ttl=30)` so manual refreshes don't hammer the filesystem.

### Catalog-specific Dos and Don'ts

✓ Keep `catalog.yaml` as the **only** place where analysis content lives — no hardcoded titles or descriptions in Python pages.
✓ Keep sketch functions **pure** (no I/O, no side effects, deterministic seeds).
✓ Update `catalog.yaml` **and** `viz_catalog/README.md` when adding a new analysis group or stage.
✓ Pin the `GITHUB_BASE` branch to `develop` so notebook links stay valid.
✓ Run `ruff check viz_catalog/` and `ruff format viz_catalog/` after any Python change.

✗ Do **not** import from `src/` inside any `viz_catalog/` file — the catalog must stay self-contained.
✗ Do **not** add heavyweight dependencies (MNE, PyTorch, SciPy) to `viz_catalog/requirements.txt`.
✗ Do **not** use `plt.show()` inside sketch functions — always return the `Figure` object and let Streamlit render it.
✗ Do **not** store actual EEG data or plot files in `viz_catalog/` — the Results Browser reads them from an external local path.

## Important Notes

- This is a **neuroscience research project** analyzing the effects of psilocybin on neural synchrony
- The primary metric is **Inter-Subject Correlation (ISC)** — the correlation of EEG signals across participants
- EEG data is high-dimensional: 257 channels × thousands of timepoints × multiple participants
- Artifact removal (ICA) is critical for data quality
- Always validate data integrity at each pipeline stage
- Metadata tracking is essential for reproducibility

---
> Source: [dbeinhauer/psilocybin-eeg](https://github.com/dbeinhauer/psilocybin-eeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
