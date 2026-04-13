## yhack-eco-bench

> A benchmark results website for displaying and comparing evaluation data. Design language uses **OpenTable design tokens** (colors, spacing, typography, shadows, breakpoints) with **Cedar** (REI) component patterns — clean, card-based layout with BrandonText typography.

# EcoBench — Benchmark Results Website

## Project Overview

A benchmark results website for displaying and comparing evaluation data. Design language uses **OpenTable design tokens** (colors, spacing, typography, shadows, breakpoints) with **Cedar** (REI) component patterns — clean, card-based layout with BrandonText typography.

## Tech Stack

- **Frontend**: Static HTML/CSS/JS (no build step, Vercel-deployable as-is)
- **Typography**: BrandonText → system font fallback stack
- **Design Tokens**: OpenTable (`opentable.github.io/design-tokens`)
- **Component Patterns**: Cedar (`cedar.rei.com`)
- **JS**: Vanilla JavaScript, zero dependencies

## File Structure

```
index.html    — Main page (nav, hero, leaderboard, categories, comparison, methodology, footer)
styles.css    — OT design tokens as CSS custom properties, Cedar component patterns, dark mode, responsive
script.js     — Interactivity, placeholder data, rendering logic
CLAUDE.md     — This file
_backup/      — Previous version backup
```

## Design Tokens (from OpenTable)

- **Colors**: Ash palette for neutrals, Red (`#da3743`) as primary accent, Green (`#2f864d`) for positive/scores, Blue (`#4a6fde`) for comparison B, Yellow for rank-1 gold
- **Spacing**: 4/8/16/32/48/64px scale
- **Border radius**: 4px
- **Shadow**: `0px 2px 4px rgba(45, 51, 63, .2)`
- **Breakpoints**: Desktop ≥1056px, Tablet 768–1055px, Mobile <768px
- **Typography**: BrandonText 700/500/400 at sizes 48/32/24/18/16/14px

## Semantic Tokens

Light/dark themes switch via `[data-theme="dark"]` on `<html>`. Semantic variables (`--color-text`, `--color-bg`, `--color-surface`, etc.) remap to appropriate OT palette values per theme.

## Data Model

Website benchmark data lives in `data.js` as `BENCHMARK_DATA` (array of experiment objects).

### Where the real data comes from

The source of truth is the SQLite databases in `yhacktemp/yhacktemp/autoresearch-yaledgx/runs/run_*/results.db`. Each DB has an `experiments` table with `config_json`, `metrics_json`, `status`, `strategy_used`, `pareto_rank`, and `created_at` columns. To re-extract fresh data, query all `results.db` files for `status='completed'`, deduplicate by config hash (keep latest), and recompute Pareto ranks on (SCI ↓, BPB ↓). The analysis script at `yhacktemp/yhacktemp/autoresearch-yaledgx/analyze_all.py` does exactly this and can export combined JSON/CSV via `--export`. A pre-exported snapshot also lives at `yhacktemp/yhacktemp/autoresearch-yaledgx/runs/combined/combined_results.json`.

## How to Run

```sh
# Any static file server works
npx serve .
python3 -m http.server 8000
# Or just open index.html directly
```

Deploys to Vercel with zero config (static files, no build step).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bdavidzhang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bdavidzhang)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
