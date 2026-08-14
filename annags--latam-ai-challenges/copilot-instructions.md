## latam-ai-challenges

> You're working inside the **WAT framework** (Workflows, Agents, Tools). This architecture separates concerns so that probabilistic AI handles reasoning while deterministic code handles execution. That separation is what makes this system reliable.

# Agent Instructions

You're working inside the **WAT framework** (Workflows, Agents, Tools). This architecture separates concerns so that probabilistic AI handles reasoning while deterministic code handles execution. That separation is what makes this system reliable.

## The WAT Architecture

**Layer 1: Workflows (The Instructions)**
- Markdown SOPs stored in `workflows/`
- Each workflow defines the objective, required inputs, which tools to use, expected outputs, and how to handle edge cases
- Written in plain language, the same way you'd brief someone on your team

**Layer 2: Agents (The Decision-Maker)**
- This is your role. You're responsible for intelligent coordination.
- Read the relevant workflow, run tools in the correct sequence, handle failures gracefully, and ask clarifying questions when needed
- You connect intent to execution without trying to do everything yourself
- Example: If you need to pull data from a website, don't attempt it directly. Read `workflows/scrape_website.md`, figure out the required inputs, then execute `tools/scrape_single_site.py`

**Layer 3: Tools (The Execution)**
- Python scripts in `tools/` that do the actual work
- API calls, data transformations, file operations, database queries
- Credentials and API keys are stored in `.env`
- These scripts are consistent, testable, and fast

**Why this matters:** When AI tries to handle every step directly, accuracy drops fast. If each step is 90% accurate, you're down to 59% success after just five steps. By offloading execution to deterministic scripts, you stay focused on orchestration and decision-making where you excel.

## How to Operate

**1. Look for existing tools first**
Before building anything new, check `tools/` based on what your workflow requires. Only create new scripts when nothing exists for that task.

**2. Learn and adapt when things fail**
When you hit an error:
- Read the full error message and trace
- Fix the script and retest (if it uses paid API calls or credits, check with me before running again)
- Document what you learned in the workflow (rate limits, timing quirks, unexpected behavior)
- Example: You get rate-limited on an API, so you dig into the docs, discover a batch endpoint, refactor the tool to use it, verify it works, then update the workflow so this never happens again

**3. Keep workflows current**
Workflows should evolve as you learn. When you find better methods, discover constraints, or encounter recurring issues, update the workflow. That said, don't create or overwrite workflows without asking unless I explicitly tell you to. These are your instructions and need to be preserved and refined, not tossed after one use.

## The Self-Improvement Loop

Every failure is a chance to make the system stronger:
1. Identify what broke
2. Fix the tool
3. Verify the fix works
4. Update the workflow with the new approach
5. Move on with a more robust system

This loop is how the framework improves over time.

## File Structure

**What goes where:**
- **Deliverables**: Final outputs go to cloud services (Google Sheets, Slides, etc.) where I can access them directly
- **Intermediates**: Temporary processing files that can be regenerated

**Directory layout:**
```
.tmp/           # Temporary files (scraped data, intermediate exports). Regenerated as needed.
tools/          # Python scripts for deterministic execution
workflows/      # Markdown SOPs defining what to do and how
.env            # API keys and environment variables (NEVER store secrets anywhere else)
credentials.json, token.json  # Google OAuth (gitignored)
```

**Core principle:** Local files are just for processing. Anything I need to see or use lives in cloud services. Everything in `.tmp/` is disposable.

## Bottom Line

You sit between what I want (workflows) and what actually gets done (tools). Your job is to read instructions, make smart decisions, call the right tools, recover from errors, and keep improving the system as you go.

Stay pragmatic. Stay reliable. Keep learning.

---

## Project: LATAM AI Governance Challenges Repository

**What it does:** Streamlit web app — browsable, searchable repository of AI governance challenges in Latin America from a systematic literature review of 38 articles (2021–2026).

**Data source:** Google Sheets (ID `1vjfeBiTcjn1m-gCoNmmBVUe-FJS-sjSUBXuYzGy47Y4`) via Sheets API v4. Service account: `ai-challenges-repo@gen-lang-client-0246752153.iam.gserviceaccount.com`. Credentials in `credentials.json` (gitignored).

**Tech stack:** Python 3.12 · Streamlit ≥1.35 · pandas · Plotly Express · Cytoscape.js 3.28 + fcose 2.2 (CDN, direct HTML) · Google API Python Client

**File map:**
```
app.py                         # Main entry point, 5 tabs
drive_connector.py             # Sheets API v4 connector (@st.cache_resource);
                                # credentials: credentials.json → st.secrets["gcp_service_account"]
                                # (Streamlit Cloud) → GOOGLE_APPLICATION_CREDENTIALS
data_loader.py                 # Cached loaders per sheet (@st.cache_data ttl=3600)
content/
  about.md                     # About tab content (rendered via Path.read_text in app.py) — edit freely
components/
  colors.py                    # Centralized CAT1_COLORS / SPECIFICITY_COLORS + badge() / to_doi_url() helpers (single source of truth)
  filters.py                   # Inline (main-body) filter widgets — no sidebar since v1.3
  challenge_card.py            # Card grid + @st.dialog modal
  charts.py                    # Plotly charts (pie, bar, horizontal bar, LATAM choropleth map)
  network_graph.py             # Cytoscape.js + fcose bipartite network (self-contained HTML).
                                # NOT used in the UI since v1.2 (Mapa de Relaciones tab removed),
                                # kept on disk for possible future reuse.
credentials.json               # GITIGNORED — service account key
```

**Sheet schema (v3, 2026-06-10):**
- `Challenges Repository`: `id, cat_1, cat_2_en, cat_2_es, description_en, description_es, regional_specificity, regional_rationale, wirtz_mapping, key_articles, article_count, first_year_cited, countries_mentioned`
  - `title_en` / `title_es` / `cat_2` REMOVED. `cat_2_en` / `cat_2_es` / `description_es` are NEW.
  - `cat_2_en` is now the challenge's display title (used in cards, detail modal, and `chart_top_challenges_by_articles`). `description_es` is not used in the UI yet.
  - `key_articles` values are unpadded `Articles Reference.id`-style refs (e.g. `A3`, `D1`). The detail
    modal normalizes these (zero-pads to `A03`/`D01` via `_normalize_article_id()` in `challenge_card.py`),
    matches against `Articles Reference.id`, and displays the matched `quick_ref` as a DOI link.
- `Articles Reference`: `id, quick_ref, database, year, author_short, Title→title, doi, language, country_focus`
  - `type` (column C) REMOVED; `database` is the new column C (not used in UI).
- `Challenges by Paper`: `paper_id, quick_ref, author_short, year, title_short, challenge_id, cat_1, cat_2_en, verbatim_from_paper, regional_specificity`
  - `quick_ref_en` REMOVED; `cat_2_en` ADDED — shown as part of the always-visible card header
    ("**{author_short} ({year})** — {cat_2_en}") in the By Paper tab (v1.3). `title_short` is no
    longer shown in the collapsed header (the full `title` from `Articles Reference`, joined on
    `quick_ref`, appears instead inside the "Full citation" expander together with the DOI).
- `Statistics`: raw display only

**Color conventions (v3):**
Centralized in `components/colors.py` — single source of truth, imported by `filters.py`,
`challenge_card.py`, `charts.py`, and `app.py` (Challenges by Paper badges).
```python
CAT1_COLORS = {"Socio-technical": "#7F77DD", "Ethical": "#00BF63", "Institutional": "#004AAD"}
SPECIFICITY_COLORS = {"High": "#ff3131", "Medium": "#fa6767", "Low": "#f69696"}  # badges, charts, sidebar pills
ARTICLE_COLOR = "#6c8ebf"  # network graph article nodes (network_graph.py, unused in UI as of v1.2)
```
Repository tab filters (`st.pills`, multi-select, in the main body since v1.3) for `cat_1` and
`regional_specificity` are colored via injected CSS (`_pills_color_css` in `filters.py`) to
match these palettes — selected = filled background + white text, unselected = outlined in
that color. CSS targets pill buttons by position (`nth-of-type`) scoped to
`.st-key-cat1_pills` / `.st-key-spec_pills` containers, matching the order of
`cat1_options`/`spec_options`. If Streamlit's internal pill `data-testid` changes in a future
version, adjust the selectors in `_pills_color_css`.

**Key gotchas:**
- Cytoscape.js node data must be `{"data": {...}}` format; styles reference props via `'data(propName)'`
- Tooltip is a custom `<div id="tooltip">` tracked via `document.addEventListener('mousemove')` — no HTML escaping issues (unlike vis.js)
- fcose layout (`cytoscape-fcose.js` via CDN) auto-registers with cytoscape when loaded; use `name: 'fcose'` in layout options
- Use `__PLACEHOLDER__` substitution (not f-strings) for embedding JSON into HTML/JS templates
- `@st.dialog` must be defined at module level, not inside a loop
- DOI values may lack `https://doi.org/` prefix — normalize on display
- DOI values may be a placeholder dash (`—`, em-dash) meaning "no DOI" — only render as a link if
  the DOI string contains an alphanumeric character, otherwise show plain text
- `Title` column in Articles sheet has capital T — renamed to `title` in `load_articles()`
- All UI navigation/labels must be in English (e.g. "View detail", not "Ver detalle")
- `st.expander()` labels don't support `unsafe_allow_html` — colored badges for an expander
  entry must be rendered via `st.markdown(..., unsafe_allow_html=True)` immediately above it.
  Pattern used in By Paper (v1.3): wrap the whole entry in `st.container(border=True)` so
  always-visible header text + badges render above a nested `st.expander` for extra detail.
- `st.dataframe` cell text-wrapping (e.g. long "title" values in Articles tab) requires BOTH
  `column_config.TextColumn(width="large")` on that column AND a `row_height` set on
  `st.dataframe(...)` — `column_config` alone does not enable wrapping in Streamlit 1.57's
  glide-data-grid.
- `country_focus` in `Articles Reference` mixes individual countries with continent-level
  entries ("Latin America", "South America") and non-LATAM countries ("Canada", "USA"). To
  build LATAM-only views, filter against the `LATAM_COUNTRIES` allowlist in
  `components/charts.py` rather than excluding specific non-LATAM values.

**Status (2026-05-28 — v1.1 complete):**
v2 UI revision fully applied:
- ✅ Modal cards (`@st.dialog`) replacing expanders in Repository tab
- ✅ New color palette: Socio-technical=#7F77DD, Ethical=#00BF63, Institutional=#004AAD
- ✅ Pie charts for category + specificity breakdowns
- ✅ Country chart (`chart_countries_cited`) in Statistics tab
- ✅ Quick Ref filters in By Paper tab
- ✅ Full article titles + quick_ref column in Articles tab; DOI normalized to https://doi.org/
- ✅ Network graph: article nodes labeled with quick_ref, colors updated, tooltips render as HTML
- ✅ "About" tab with methodology + changelog
- ✅ Network graph: migrado de vis.js → Cytoscape.js 3.28 + fcose layout (v1.2, 2026-05-28)

**Status (2026-06-10 — v1.2 / Schema v3 migration complete):**
Google Sheet restructured (new headers across all 3 data sheets, see "Sheet schema (v3)"
above). App updated to match, plus UI/color/language cleanup:
- ✅ `data_loader.py`: column-strip lists updated for the new headers
- ✅ `components/colors.py` (NEW): centralized `CAT1_COLORS` + `SPECIFICITY_COLORS`
  (High=#ff3131, Medium=#fa6767, Low=#f69696) + shared `badge()` helper
- ✅ Sidebar filters (`filters.py`): `cat_1` / `regional_specificity` now use
  `st.pills(selection_mode="multi")` colored via CSS to match badge colors
- ✅ `cat_2_en` is now the challenge title throughout (cards, detail modal, top-10 chart)
- ✅ Detail modal "Key articles" now resolve to clickable `quick_ref` + DOI links (zero-pad
  normalization to match `Articles Reference.id`; placeholder `—` DOIs render as plain text)
- ✅ "Ver detalle" → "View detail" (all navigation now in English)
- ✅ Articles tab: "Type" filter removed (column gone); display_cols updated
- ✅ "Challenges by Paper" tab restructured: colored cat_1/specificity badges above each
  expander, `quick_ref_en` removed, "**Challenge:**" now shows `cat_2_en`
- ✅ About tab content moved to `content/about.md` (editable without touching app.py)
- ✅ "Mapa de Relaciones" tab removed from UI (6 tabs → 5); `network_graph.py`/`lib/`
  retained on disk, unused

**Status (2026-06-10 — v1.3 layout revision complete):**
- ✅ Sidebar removed entirely. Repository tab filters (`AI challenges category`,
  `regional specificity` pills + `Search`) moved to the main body via
  `render_repository_filters()` in `filters.py` (renamed from `render_sidebar_filters`),
  laid out in `st.columns` like the By Paper inline filters; same pills+CSS coloring,
  `_pills_color_css` unchanged. "Refresh data" button moved into this row.
  `st.set_page_config(initial_sidebar_state=...)` and the `[data-testid="stSidebar"]` CSS
  rule were removed (no longer used anywhere).
- ✅ By Paper tab restructured: each entry is now an `st.container(border=True)` card whose
  always-visible header shows `**{author_short} ({year})** — {cat_2_en}` plus the `cat_1` /
  `regional_specificity` badges (previously rendered above the expander). A nested
  `st.expander("Full citation")` shows the full citation — author, year, full `title` and DOI
  link (via new `to_doi_url()` in `components/colors.py`), obtained by merging
  `Challenges by Paper` with `Articles Reference[["quick_ref","title","doi"]]` on
  `quick_ref` — plus the existing verbatim excerpt block.
- ✅ Statistics: `chart_countries_cited` (`charts.py`) now filters `country_focus` through a
  new `LATAM_COUNTRIES` allowlist (excludes "Latin America"/"South America"/"Canada"/"USA")
  and is retitled "LATAM countries cited in the corpus". New `map_countries_cited()` renders
  the same filtered counts as a `px.choropleth` (`locationmode="country names"`,
  `fitbounds="locations"`) shown alongside the bar chart. Shared counting logic extracted to
  `_latam_country_counts()`.
- ✅ `chart_top_challenges_by_articles`: legend moved below the chart
  (`legend=dict(orientation="h", y=-0.2, x=0.5, xanchor="center")`, `margin=dict(b=90)`).
- ✅ Articles tab: `title` column now uses `column_config.TextColumn(width="large")` +
  `st.dataframe(..., row_height=80)` to wrap long titles instead of horizontal scroll.

**Status (2026-06-10 — deployment prep complete):**
- ✅ `drive_connector.py`: `_get_credentials()` now also checks
  `st.secrets["gcp_service_account"]` (between the existing `credentials.json` and
  `GOOGLE_APPLICATION_CREDENTIALS` checks), via
  `service_account.Credentials.from_service_account_info(dict(st.secrets["gcp_service_account"]), ...)`.
  Wrapped in `try/except FileNotFoundError` since `st.secrets` raises
  `StreamlitSecretNotFoundError` (a `FileNotFoundError` subclass) when no
  `secrets.toml` exists at all — this is the path that makes Streamlit Community
  Cloud deploys work as documented in README.md.
- ✅ `requirements.txt`: removed unused `openpyxl`, `networkx`, `pyvis` (no Excel
  fallback or vis.js/pyvis usage in the current codebase).
- ✅ `.gitignore`: added `.claude/`, `test_connection.py`, `test_data.py`, `lib/`
  (dev-only scratch files / superseded vis.js assets — not part of the deployed app).
- ✅ `README.md`: updated credentials-priority docs, removed the never-implemented
  `data/*.xlsx` local-fallback section, refreshed "Project structure" and "Data
  source" (Google Sheets API v4, not an Excel download) and the "Refresh data"
  button location (Repository tab main body, not sidebar).

---
> Source: [annags/latam-ai-challenges](https://github.com/annags/latam-ai-challenges) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
