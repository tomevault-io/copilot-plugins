## ai-oss-impact

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Python data analysis project measuring observable changes in open source development patterns before/after AI-assisted coding adoption (Oct 2022). Collects GitHub API data from 20-25 major repositories, stores in SQLite, and produces interactive Plotly visualizations.

## Tech Stack

- Python 3.13 (managed via uv)
- PyGithub for GitHub REST API (metadata, commits)
- GitHub GraphQL API for PRs/Issues (~70% fewer API calls)
- pydantic/pydantic-settings for data validation and config
- SQLite for storage
- pandas for data manipulation
- Plotly/matplotlib for charts
- Jupyter notebooks for analysis
- ruff for linting/formatting
- pre-commit + commitizen for code quality

## Project Structure

```
src/
  config.py           # Settings (pydantic-settings), constants, repo clusters
  models.py           # Pydantic data models (RepoInfo, CommitsMonthly, PRsMonthly)
  storage.py          # SQLite operations (init, upsert, query)
  collector.py        # Main collection orchestration
  git_collector.py    # Local git operations (commits, releases from clones)
  graphql_collector.py # GitHub GraphQL API for PRs/Issues
  charts.py           # Plotly visualization functions
  metrics.py          # Statistical analysis functions (35+ functions)
  analytics/          # Dashboard analytics layer (UI-agnostic)
    data_service.py   # Data loading, caching, RepoData dataclass
    repo_analytics.py # Single-repo analysis functions (cached)
    cohort_analytics.py # Multi-repo/cluster aggregations (cached)
    comparison.py     # Repo vs repo, cluster vs cluster comparisons
    derived_metrics.py # Computed metrics (merge rate, churn, etc.)
    caching.py        # LRU cache utilities (@cohort_cached decorator)
dashboard/
  app.py              # Streamlit entry point
  pages/              # Dashboard pages (overview, repo explorer, etc.)
  components/         # Reusable UI components (filters, charts, KPIs)
.streamlit/
  config.toml         # Theme colors, server settings
notebooks/
  01_explore_single_repo.ipynb  # Phase 2 single-repo exploration
  02_cohort_overview.ipynb      # Phase 3 cohort validation
scripts/
  collect_all.py      # Cohort collection CLI (two-sweep)
  validate_data.py    # Data quality validation CLI
  generate_summary.py # Generate findings_summary.json
data/                 # SQLite database (gitignored)
repos/                # Git clones (gitignored)
output/charts/        # Exported visualizations
```

## Commands

```bash
# Setup
uv sync --all-extras
uv run pre-commit install
uv run pre-commit install --hook-type commit-msg

# Two-sweep collection (recommended)
uv run python scripts/collect_all.py --sweep git   # Fast: clone repos, extract commits/releases
uv run python scripts/collect_all.py --sweep api   # Slow: fetch PRs/issues via GraphQL
uv run python scripts/collect_all.py --sweep all   # Both sweeps (default)

# Other collection options
uv run python scripts/collect_all.py --dry-run           # Preview repos
uv run python scripts/collect_all.py --force-refresh     # Re-fetch all data
uv run python scripts/collect_all.py --repo owner/repo   # Collect specific repo
uv run python scripts/collect_all.py --stale-days 30     # Re-collect repos not updated in 30 days

# Data validation
uv run python scripts/validate_data.py           # Run all checks
uv run python scripts/validate_data.py --verbose # Include monthly gap details

# Analysis
uv run python scripts/generate_summary.py        # Generate findings_summary.json
uv run jupyter notebook notebooks/03_statistical_tests.ipynb  # Interactive analysis

# Interactive Dashboard
uv run streamlit run dashboard/app.py            # Launch dashboard at http://localhost:8501

# Lint/format
uv run ruff check .
uv run ruff format .

# Run tests
uv run pytest
```

## Key Concepts

- **Primary breakpoint**: 2022-10 (Copilot GA + ChatGPT launch)
- **Analysis window**: 2020-01 to 2025-12
- **Monthly aggregation**: All metrics stored at month granularity (YYYY-MM)
- **AI keywords**: defined in `src/config.py` - copilot, chatgpt, gpt-4, claude, cursor, etc.

## Data Model

Core tables: `repos`, `commits_monthly`, `prs_monthly`, `issues_monthly`, `releases`, `contributors`

All monthly tables have `repo_id` FK and `month` (YYYY-MM format).

The `repos` table tracks collection status:
- `collection_status`: pending | git_in_progress | git_complete | api_in_progress | completed | failed
- `last_error`: Error message for failed collections
- `last_api_collected_at`: Timestamp of last successful API sweep (for stale detection)

### Two-Sweep Collection Flow

```
SWEEP 1 - Git (fast, minimal API):
  pending → git_in_progress → git_complete
  - Clone repos locally (shallow since 2020)
  - Extract commits from git log
  - Extract releases from git tags

SWEEP 2 - API (GraphQL, rate-limited):
  git_complete → api_in_progress → completed
  - Fetch PRs via GraphQL (100 items/page)
  - Fetch issues via GraphQL (100 items/page)
  - Build contributors from PR data
```

### Checkpointing

Collection progress is saved after each GraphQL page:
- `collection_progress` table stores cursor + full state
- On interruption: resume from exact cursor position
- Worst case data loss: ~100 items (1 page)

## Environment

Requires `GITHUB_TOKEN` in `.env.local` file for API access.

Optional overrides in `.env.local`:
- `TEST_REPO` - Override default test repo (e.g., `hukkin/tomli`)
- `DATE_RANGE_START` - Override start date (e.g., `2022-01`)
- `DATE_RANGE_END` - Override end date (e.g., `2024-06`)
- `COLLECT_REPOS` - Comma-separated list of repos for cohort collection (e.g., `hukkin/tomli,psf/requests`). If not set, uses all repos in `CLUSTER_MAP`.

## GitHub API Usage

### Rate Limits
- REST API: 5,000 requests/hour
- GraphQL API: 5,000 points/hour (queries cost ~1 point each)

### API Calls per Data Type (with GraphQL)

| Data Type | Method | API Calls | Notes |
|-----------|--------|-----------|-------|
| Repo metadata | REST | 1 | Single call |
| Commits | Git | 0 | Extracted from local clone |
| Releases | Git | 0 | Extracted from git tags |
| PRs | GraphQL | N/100 | 100 items/page (was N/30 with REST) |
| Issues | GraphQL | N/100 | 100 items/page (was N/30 with REST) |
| Contributors | - | 0 | Derived from PR data |

### Estimates per Repository Size (with GraphQL)

| Size | PRs | Issues | Est. API Calls | Est. Time |
|------|-----|--------|----------------|-----------|
| Small | 200 | 100 | ~5 | ~1 min |
| Medium | 2,000 | 1,000 | ~30 | ~5 min |
| Large | 30,000 | 10,000 | ~400 | ~30 min |

### Full Cohort (21 repos)
- With GraphQL: ~3,000-5,000 API calls total
- At 5,000/hour limit: **~1-2 hours for API sweep**
- Git sweep runs in parallel, no rate limiting

### Caching
- Git clones cached in `repos/` directory
- Default `max_age_days=7` - clones updated if older
- SQLite data cached with `collected_at` timestamp
- API data tracked with `last_api_collected_at` - use `--stale-days N` to re-collect
- `force_refresh=True` bypasses cache entirely

### Monitoring
```python
from src.collector import collect_repo_git_only, collect_repo_api_only

# Git sweep (fast)
result = collect_repo_git_only("owner/repo")
print(f"Commits: {result['commits_count']}, Releases: {result['releases_count']}")

# API sweep (GraphQL)
result = collect_repo_api_only("owner/repo")
print(f"PRs: {result['prs_count']}, Issues: {result['issues_count']}")
```

## Adding a New Repository

1. Edit `src/config.py` - add to `CLUSTER_MAP`:
   ```python
   CLUSTER_MAP = {
       ...
       "owner/repo": "cluster-name",
   }
   ```

2. Run collection:
   ```bash
   uv run python scripts/collect_all.py --sweep git --repo owner/repo
   uv run python scripts/collect_all.py --sweep api --repo owner/repo
   ```

3. Validate: `uv run python scripts/validate_data.py`

4. Re-run analysis: `uv run python scripts/generate_summary.py`

## Analysis Workflow

### Statistical Analysis (metrics.py)

Key functions:
- `run_full_analysis(repo_id, commits_df, prs_df, issues_df, breakpoint)` - Full pre/post comparison
- `test_normality(data)` - Shapiro-Wilk test
- `compare_periods(pre, post)` - t-test or Mann-Whitney U based on normality
- `calculate_ai_mention_rate(commits_df, prs_df)` - AI keyword tracking
- `detect_seasonality(df, value_col, period=12)` - Autocorrelation analysis

### Visualization (charts.py)

Key functions:
- `create_effect_size_heatmap(cohort_df, repo_names)` - Effect sizes across repos
- `create_correlation_heatmap(corr_matrix)` - Metric correlations
- `create_trend_comparison_chart(df, value_col, breakpoint)` - Pre/post trends
- `create_cohort_overview_chart(data, metric, title)` - Cohort summary

### Output Files

- `output/findings_summary.json` - Machine-readable analysis results
- `output/charts/*.html` - Interactive Plotly visualizations

## Interactive Dashboard

A Streamlit dashboard provides interactive exploration of the analysis results.

### Dashboard Pages

1. **Overview** - Cohort-level KPIs, effect size heatmap, top findings
2. **Repo Explorer** - Single repo deep-dive with time series and statistical tests
3. **Cluster Analysis** - Within/across cluster comparisons
4. **Statistical Analysis** - Full statistical exploration with configurable alpha
5. **AI Adoption** - AI mention tracking, rankings, correlations
6. **Data Model** - ERD diagram, table schemas, metric documentation

### Breakpoint Presets

The dashboard supports multiple breakpoints for comparison:
- Copilot GA + ChatGPT (2022-10)
- GPT-4 (2023-03)
- GPT-4 Turbo (2023-11)
- Claude 3 Opus (2024-03)
- Claude 3.5 Sonnet (2024-06)
- Claude 3.5 Sonnet v2 (2024-10)
- Custom dates

### Analytics Module (`src/analytics/`)

UI-agnostic analytics functions that can be reused outside Streamlit:

```python
from src.analytics import load_repo_data, get_repo_summary, compare_repos

# Load repo data
repo_data = load_repo_data(repo_id)
summary = get_repo_summary(repo_data)

# Compare repos
comparison = compare_repos(repo_a, repo_b, metrics=("commits", "prs_merged"))  # tuple for caching
```

### Caching Architecture

Two-layer caching strategy for performance:

| Layer | Decorator | Use Case |
|-------|-----------|----------|
| Analytics (business logic) | `@lru_cache` | Single-repo functions (hashable RepoData) |
| Analytics (cohort) | `@cohort_cached` | Functions taking dict[int, RepoData] |
| Dashboard (UI/IO) | `@st.cache_data(ttl=3600)` | Page-level data loading |

The `@cohort_cached` decorator converts unhashable dict arguments to hashable tuples:

```python
from src.analytics.caching import cohort_cached, clear_all_analytics_caches

@cohort_cached(maxsize=16)
def get_effect_size_matrix(cohort_data: dict[int, RepoData], breakpoint: str) -> pd.DataFrame:
    ...

# Clear all caches globally
clear_all_analytics_caches()
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kotaicode)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kotaicode)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
