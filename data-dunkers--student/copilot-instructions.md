## student

> Repository for student-facing Jupyter notebook activities and supporting HTML exports for the Data Dunkers program.

# Data Dunkers Student Portal

Repository for student-facing Jupyter notebook activities and supporting HTML exports for the Data Dunkers program.

## What’s Here

- `activities/` — primary student notebooks grouped by topic (interpreting visuals, data skills, applied analysis) plus `HTML-Exports/` for rendered copies (ignored in git).
- `notebooks/` — challenge and enrichment notebooks using current basketball datasets.
- `docs/` — simple static index page that links to many lesson variants.
- `requirements.txt` — minimal Python stack (Jupyter, pandas/numpy, plotly.express, requests/bs4/lxml).
- `.venv/` (optional) — local virtual environment if you create one.

## Getting Started

1) Install Python 3.10+ and create a virtual environment: `python -m venv .venv && .\.venv\Scripts\activate` (Windows) or `source .venv/bin/activate` (macOS/Linux).  
2) Install dependencies: `pip install -r requirements.txt`.  
3) Launch Jupyter: `jupyter lab` (or `jupyter notebook`) and open files in `activities/` to work through lessons top-to-bottom.

## Tips for Using the Notebooks

- Keep outputs and written reflections in-place; duplicate files if you want a fresh copy.
- Run cells sequentially; most notebooks assume earlier setup cells have executed.
- Visual notebooks (using `plotly.express` exclusively) focus on reading charts; data skill notebooks cover cleaning, sorting/merging, correlations, descriptive stats, and simple synthetic data; applied analysis includes shot charts.
- Challenge notebooks in `notebooks/` explore live or recent NBA data (player profiles, shot selection, partial-season trends, etc.) and may require network access for API calls.

## Updating or Publishing

- Export HTML versions of notebooks into `activities/HTML-Exports/` to keep outputs alongside source without committing them.
- If you host the static site, point to `docs/index.html` or generate fresh links after adding new lessons.

## Index File Management

To manage the progression of lessons as they are developed, we use two separate index files in `docs/`:

- **`index2.html` (Master Backup):** This is the master index. It contains the full list of all lessons, including those marked as *(forthcoming)*. It should be used as the primary source of truth for the complete curriculum.
- **`index.html` (Student View):** This is a pruned version of the master index. It only displays lessons that are finished and ready for students.

**Important:** Any structural changes, description updates, or new lesson additions must be applied to **both** files to ensure consistency between the internal roadmap and the external student portal.

## Lesson Page Template Requirements

To ensure consistency across the student portal, all lesson pages in `docs/` must follow the standardized format established in `docs/shot-charts.html`.

### 1. Visual Branding & Layout

- **Banners:** Include top and bottom banners from the Data-Dunkers repository.
- **Header:** Use an `<h2>` for the title and a brief introductory paragraph.
- **Styling:** Standard 2% margins and `sans-serif` font.

### 2. Interactive Visualization

- **Iframe:** Embed the Plotly visualization from `docs/visualizations/` using an `<iframe>`.
- **Interaction Rules:** Provide a short explanation of how to use the visualization (hover, legend filters, etc.).

### 3. Critical Thinking Questions

- **Dynamic Rendering:** Define questions in a JS array and render them as `<label>` and `<input>` elements.
- **Persistence:** Use `localStorage` to save answers automatically. **Important:** The storage key must be unique to the page (e.g., `filename + 'Answers'`) to avoid overwriting data between lessons.
- **Copy Feature:** Include a "Copy Answers" button that formats the Q&A for the clipboard.

### 4. Navigation & Links

- **Tooling:** Include dynamic links to the source notebook on GitHub, Google Colab, and Callysto Hub.
- **Page Links:** Provide clear "home" and "next" lesson links in the footer.

## Support

- Issues typically stem from missing packages or blocked network calls for live-data notebooks. Re-run `pip install -r requirements.txt`, and if fetching data fails, retry on a different network or mock data locally.

## Style Guide

For the most up-to-date standards, please refer to the [Master Style Guide](https://docs.google.com/document/d/1oik8ZycHxHAwmqyespPV3a0997yFidHJrqPZTytj9zQ/edit?tab=t.0).

### Jupyter Notebooks & Code

- **Imports**: Library imports should be at the top of the first code cell.
- **Outputs**: Every code cell should output something (even if specific print statements like `print('Libraries imported')`).
- **Execution Flow**: Cells must run correctly from top to bottom.
- **Comments**: Avoid comments in code cells unless absolutely necessary; use Markdown cells for explanations.
- **Quotes**: Prefer single quotes (`'`) over double quotes (`"`) in Python code.
- **Cell Size**: Prefer short code cells over long ones.
- **Data Inspection**: Always use `df.head()` immediately after importing data (rather than just `df`).
- **Variables**: We don’t always need a `fig` variable.
- **Standard Variable Names**: Use `df = pd.read_csv(url)` for consistency.
- **Space for Automation**: You may leave a blank cell at the top and bottom of a notebook for scripts to inject banners and links.

### Visual Branding & Navigation

- **Banners**: Include a banner at the top of each notebook and webpage.
  - Image URL: `https://github.com/Data-Dunkers/lessons/blob/main/images/top-banner.jpg?raw=true`
- **Online Access**: Include "Open in Colab" and "Open in Callysto" links at the bottom of each notebook (these can be automated).

### Git & Naming Conventions

- **Commit Messages**: Should be descriptive and in the imperative voice (e.g., "Fix library imports").
- **File Names**: Use lowercase and kebab-case (e.g., `interpreting-scatterplots.ipynb`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Data-Dunkers)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Data-Dunkers)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
