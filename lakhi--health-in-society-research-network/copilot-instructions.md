## health-in-society-research-network

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Visualisation project for the Health in Society Research Network at the University of Vienna. Processes survey data from network members to produce word clouds, maps, and alluvial/Sankey plots, bundled into a self-contained HTML output page.

## Running the Project

There are no build or test commands. The workflow is notebook-based:

1. Open notebooks in `/notebooks/` using Jupyter
2. Each notebook reads from `data/raw/GiG2025_clean.csv` via `load_and_clean_survey_data()`
3. Outputs (`.png` word clouds, `.html` plots) are saved to `data/raw/` (processed outputs live alongside raw data)
4. The final HTML page (`gig2025_visualisations_2_de.html` / `gig2025_visualisations_2.html`) is assembled in `template_start.ipynb`

Run a single notebook non-interactively:
```bash
jupyter nbconvert --to notebook --execute notebooks/<notebook_name>.ipynb
```

## Architecture

### Data Flow
```
data/raw/GiG2025_clean.csv
    → src/data_processing.py (GiGSurveyDataProcessor)
    → notebooks/ (one per visualisation type)
    → data/raw/ (output assets: .png, .html)
    → template_start.ipynb (final HTML assembly)
```

### `src/data_processing.py`
Central cleaning module. `GiGSurveyDataProcessor` handles:
- **Department normalisation**: prefixes bare names with `"Department of "`, fixes known edge cases
- **Collaboration columns**: splits `collab_faculties` (3 max) and `collab_unis` (11 max) from `+`-delimited strings into individual columns
- **Keyword columns**: splits `health_research_keywords` and `research_methods_keywords` on commas/semicolons into lists of stripped strings

All notebooks import via:
```python
sys.path.append(os.path.abspath(os.path.join('..', 'src')))
from data_processing import load_and_clean_survey_data
```

### Notebooks
| Notebook | Purpose |
|---|---|
| `word_clouds.ipynb` | Generates word clouds from health/methods keywords using `wordcloud` + matplotlib; includes keyword normalisation dictionaries (replacements defined inline in the notebook) |
| `maps.ipynb` | Choropleth/collaboration maps using Plotly + shapefiles in `data/raw/ne_10m_admin_*` |
| `alluvial_plots.ipynb` | Sankey/alluvial diagrams of collaborations using Plotly |
| `exploratory.ipynb` | Ad-hoc exploration |
| `template_start.ipynb` | Assembles final HTML output page |

### Keyword Normalisation
Keyword replacement dictionaries live **inside the notebooks** (not in `data_processing.py`). Two sets exist:
- `health_keyword_replacements` in `word_clouds.ipynb` — maps variants to canonical health keywords
- `rm_replacements` in `word_clouds.ipynb` — same for research methods keywords

The sentinel value `-99` represents missing/NA responses in the raw CSV.

### Output Assets
- Word cloud PNGs use a custom blue colormap (`custom_blues`) derived from `plt.cm.Blues` with a minimum lightness floor to avoid near-white text
- Word clouds can optionally use `uni.png` (University of Vienna logo) as a mask shape
- Plotly visualisations are exported as standalone HTML fragments embedded in the final page
- Final HTML exists in both English (`gig2025_visualisations_2.html`) and German (`gig2025_visualisations_2_de.html`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lakhi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lakhi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
