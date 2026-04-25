## gvtc-budget-dashboard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About Me

- Spencer Winegarden, digital strategy leader in telecom (GVTC Communications)
- Based in Texas Hill Country, timezone America/Chicago
- Highly technical, systems thinker, AI-forward

## Communication Style

- Direct, no fluff. Lead with the answer, expand only when depth earns it.
- Recommendations over options. Say what you'd do and why.
- No emdashes. No buzzwords ("leverage", "streamlined", "cutting-edge").
- Challenge me when a better path exists. Don't hedge with "it depends" when you have a position.
- No sycophancy. Never open with "Great question!" or "Absolutely!"
- Conversational but professional. Uses contractions naturally.

## Preferences

- Write clean, well-commented code
- Prefer simple solutions over clever ones
- Explain what you changed and why after edits
- Don't ask permission for obvious next steps, just do them
- If something doesn't make sense, say so

## Project Overview

GVTC Department 44 (Web Management) budget dashboard — a single-file Streamlit app that pulls budget forecast and monthly variance report spreadsheets from Google Drive, parses them, and renders interactive Plotly charts and tables.

## Running the App

```bash
pip install -r requirements.txt
streamlit run app.py
```

The app runs on port 8501. A devcontainer config is included for Codespaces (Python 3.11).

## Secrets Configuration

The app requires a `.streamlit/secrets.toml` file (gitignored) with:

```toml
[google_drive]
refresh_token = "..."
client_id = "..."
client_secret = "..."
token_uri = "https://oauth2.googleapis.com/token"
folder_id = "..."
```

## Architecture

Everything lives in `app.py` — no modules or packages. The file is structured as:

1. **Configuration** — GL account mappings (`GL_NAMES`), Plotly theme (`PLOTLY_LAYOUT`), month labels
2. **Parsers** — `parse_forecast_file()` handles the multi-year budget forecast xlsx; `parse_variance_report()` handles monthly actual-vs-budget xlsx. Both use `xlsx2csv` to convert xlsx bytes to CSV rows, then parse positionally by column index.
3. **Google Drive integration** — `load_drive_data()` fetches all files from a Drive folder, classifies them by filename keywords ("variance"/"actual" → variance report, "budget"/"forecast" → forecast), parses each, and merges results. Cached with `@st.cache_data(ttl=300)`.
4. **Sidebar** — navigation radio (4 pages) and category multiselect filter
5. **Pages** — Budget Overview, Monthly Actuals, Variance Analysis, Year Comparison. Each page filters by `filtered_accounts` from the sidebar.

## Key Conventions

- Dollar amounts are plain floats (not cents). Parenthetical values like `(1,265)` are parsed as negative by `_to_float()`.
- GL accounts are strings like `"6124.32"`. Friendly names come from the `GL_NAMES` dict.
- The dark theme colors are hardcoded: `#2ED573` (green/actual), `#5B8DEF` (blue/budget), `#FC5C65` (red/over-budget).
- Variance is positive when under budget (budget − actual > 0).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spencerjw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
