## pro-visualize-2-0

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Session Usage Policy

**If context usage reaches 90%:** immediately (1) update `PRODUCTION_READINESS_PLAN.md` with completed work, (2) save relevant memories, then (3) stop. Do not continue implementing beyond this point. The next session will resume from the plan.

## Overview

Pro-Visualize is a Streamlit-based proteomics data visualization application supporting QC, Quantification, Comparative Analysis, Pathway Enrichment, Dilution Series, and Functional Annotation workflows.

## Running the Application

```bash
# Install dependencies
pip install -r requirements.txt

# Run the Streamlit app
streamlit run app.py
```

The application runs on port 8501 by default and opens automatically in your browser.

## Architecture

### Core Structure

The application follows a modular, tab-based architecture where each analysis type is self-contained:

- **`app.py`**: Main entry point that configures Streamlit and creates top-level tabs
- **`modules/`**: Analysis modules, each with a `render()` function called from app.py
  - Each module defines its own UI/UX and manages session state
  - Module pattern: Class-based with `_upload_data_section()`, `_display_results()`, and `render()` methods
- **`visualizations/`**: Visualizer classes that handle data processing and plot generation
  - Separated from UI logic to maintain clean architecture
  - Each visualizer is initialized with data and column configuration
  - Methods return Plotly/Matplotlib figures or BytesIO buffers
- **`utils/`**: Shared utilities
  - `helpers.py`: Decorators like `@handle_plotting_errors`
  - `caching.py`: Caching functions for expensive operations
  - `data_manager.py`: Data handling utilities
- **`config/`**: Configuration files (currently minimal)

### Module ↔ Visualizer Relationship

Each module instantiates its corresponding visualizer class and stores it in `st.session_state`:

```
modules/quant_module.py (QuantificationTab)
    ↓ creates & stores in session_state
visualizations/quant_visualizer.py (QuantificationVisualizer)
    ↓ returns figures to
modules/quant_module.py
```

**Pattern used throughout:**
1. User uploads data via module UI
2. Module validates input and creates Visualizer instance
3. Visualizer stored in `st.session_state.{module_name}_visualizer`
4. Module displays plots by calling visualizer methods

### Column Configuration Pattern

Modules use configurable column names to support diverse proteomics file formats:

```python
# Users specify column names via text inputs:
protein_col = st.text_input("Protein ID Column", value="Protein")
sample_col = st.text_input("Sample Linking Column", value="Level3")
group_col = st.text_input("Group Column", value="attribute_ExperimentalGroup")

# These are passed to the visualizer constructor:
visualizer = QuantificationVisualizer(
    protein_df, annotation_df,
    protein_col=protein_col,
    sample_col=sample_col,
    group_col=group_col
)
```

For comparative analysis, a `column_config` dict is used instead:

```python
column_config = {
    "protein_id": protein_id_col,
    "sample_id": sample_id_col,
    "fold_change": fold_change_col,
    "fdr": fdr_col,
    ...
}
visualizer = ComparativeVisualizer(protein_df, annotation_df, comparative_df, column_config)
```

### Session State Management

- Each analysis module maintains its own session state namespace
- Pattern: `st.session_state.{module}_visualizer` stores the visualizer instance
- Comparative module also uses `st.session_state.significant_proteins` to share filtering results across tabs
- Enrichment results cached in `st.session_state.enrichment_results`

### Data Flow

1. **File Upload**: Users upload CSV/TSV/TXT files (auto-detected separator with `sep=None, engine='python'`)
2. **Data Validation**: Visualizer `__init__` validates required columns exist
3. **Processing**: Visualizer methods process data (filtering, transformations, clustering, etc.)
4. **Visualization**: Methods return Plotly figures, Matplotlib figures, or BytesIO buffers
5. **Display**: Module renders using `st.plotly_chart()`, `st.pyplot()`, or `st.image()`

### QC Module Structure

The QC module has a unique sub-tab architecture:

```
modules/qc_module.py
    ├── modules/qc_tabs/dia_qc_tab.py (DiaQcTab)
    │   └── visualizations/DiaQcVisualizer.py
    └── modules/qc_tabs/targeted_qc_tab.py (TargetedQcTab)
        └── visualizations/targettedQCVisualization.py
```

## Key Implementation Details

### Plot Generation Patterns

**Interactive Plotly plots** (quantification, comparative):
```python
def plot_something(self):
    fig = px.scatter(data, x='col1', y='col2', color='group')
    fig.update_layout(height=600, title="Plot Title")
    return fig
```

**Static Matplotlib plots** (heatmaps, dendrograms):
```python
def plot_heatmap(self):
    fig, ax = plt.subplots(figsize=(10, 8))
    # ... plotting logic ...
    buf = BytesIO()
    plt.savefig(buf, format="png", bbox_inches="tight")
    buf.seek(0)
    plt.close(fig)
    return buf
```

### Error Handling

Use the `@handle_plotting_errors` decorator for visualizer methods to gracefully handle exceptions in the UI:

```python
from utils.helpers import handle_plotting_errors

@handle_plotting_errors
def _display_results(self):
    # ... plotting code ...
```

### Custom Features

**Transcription Factor Highlighting**: `QuantificationVisualizer` and `ComparativeVisualizer` include a built-in set of human transcription factor gene names (`HUMAN_TRANSCRIPTION_FACTORS`) used for protein highlighting in rank-order and volcano plots.

**Protein Selection UI Pattern**: Recent modules use interactive dataframes for protein selection:
```python
selection = st.dataframe(
    protein_info_df,
    use_container_width=True,
    hide_index=True,
    on_select="rerun",
    selection_mode="multi-row"
)
selected_indices = selection.selection["rows"]
```

**Pathway Enrichment**: Uses `gprofiler-official` API with results cached in session state. The comparative module includes a comprehensive Manhattan plot and per-source dotplots.

### Clustering & Dimensionality Reduction

Pattern used in `QuantificationVisualizer._prepare_data_for_clustering()`:
1. Drop entirely empty samples
2. Impute missing values (SimpleImputer with mean strategy)
3. Standardize (StandardScaler)
4. Transpose for PCA (samples as rows, proteins as features)
5. Return scaled data and valid sample list

## Development Guidelines

### Adding a New Module

1. Create `modules/your_module.py` with a class following the established pattern
2. Create `visualizations/your_visualizer.py` with data processing and plotting logic
3. Import and add tab in `app.py`:
   ```python
   from modules.your_module import render as render_your_module
   # In tabs list: "📊 Your Module Name"
   with tab_your_module:
       render_your_module()
   ```
4. Implement `render()` function that instantiates your module class and calls `.render()`

### Adding a New Plot Type

1. Add method to appropriate visualizer class
2. Return Plotly figure, Matplotlib figure, or BytesIO buffer
3. Call from module's `_display_results()` method within appropriate tab
4. Wrap plotting calls in `try/except` or use `@handle_plotting_errors`

### File Column Name Conventions

Common column names across proteomics files:
- **Protein ID**: `Protein`, `ProteinIds`, `Accession`
- **Gene Name**: `Gene Name`, `Gene`, `Gene names`
- **Sample Linking**: `Level3`, `Sample`, `Run`
- **Experimental Group**: `attribute_ExperimentalGroup`, `Condition`, `Group`
- **Fold Change**: `log2FC`, `Log2FoldChange`
- **Significance**: `Imputed.FDR`, `FDR`, `adj.P.Val`, `p.value`

Always use configurable column names via text inputs rather than hardcoding.

## Dependencies

Key libraries (from requirements.txt):
- **UI**: streamlit (1.36.0)
- **Data**: pandas (2.2.2), numpy (1.26.4)
- **Visualization**: plotly (5.22.0), matplotlib (3.9.0), seaborn (0.13.2)
- **ML/Clustering**: scikit-learn (1.5.0), umap-learn (0.5.6)
- **Bioinformatics**: gprofiler-official (1.0.0)
- **Specialized Plots**: upsetplot (0.9.0), venn (0.1.3), pyvis (0.3.2)
- **NLP**: sentence-transformers (2.7.0), openai (1.3.7)

## Git Workflow

Main branch: `main`
Development branch: `develop`

Recent commit messages follow a conventional style:
- `feat(module): Description` for new features
- `fix: Description` for bug fixes
- Descriptive present-tense messages

## Common Issues

**Missing values**: The app replaces `0` with `np.nan` in many visualizers. Ensure this is appropriate for your data.

**Sample name mismatches**: The annotation file's sample linking column must exactly match column names in the protein data file.

**Empty samples**: Recent fixes handle samples with all missing data by dropping them before clustering/PCA to prevent shape mismatches.

---
> Source: [ajay16bharadwaj/Pro-Visualize-2.0](https://github.com/ajay16bharadwaj/Pro-Visualize-2.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
