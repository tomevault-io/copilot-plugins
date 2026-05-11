## copiloto-pauta

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file Meta Ads monitoring dashboard (`index.html`). No build step, no dependencies, no framework — open directly in a browser.

## Architecture

Everything lives in `index.html` in three sections:

- **CSS** (`<style>`): CSS custom properties in `:root` for the full color palette (dark theme, status colors). Layout uses CSS Grid on `.account-row` with 6 named columns. Responsive breakpoint at 1100 px.
- **State + data** (`<script>`): A single `state` object holds `accounts[]` (mock data) and `expanded` (a `Set` of open account IDs). Mutating state then calling `render()` is the only update pattern — there is no virtual DOM.
- **Render pipeline**: `render()` → `renderAccount(a)` → `renderTable(a)`. Each call rebuilds `innerHTML` for `#app` and re-attaches all event listeners. `event.stopPropagation()` guards on budget inputs and KPI selects prevent the row-click toggle from firing.

## Key logic

| Function | What it does |
|---|---|
| `projected(a)` | Extrapolates month-end spend: `(spent / DAY_NOW) * DAYS_MONTH` |
| `status(a)` | Returns `'green'`/`'yellow'`/`'red'` based on `projected/budget` ratio (thresholds: >1.12 → red, >0.95 → yellow) |
| `kpiValue(totals, kpi, currency)` | Computes CPA, ROAS, CTR, CPM, CPL, or CPC from campaign totals |
| `fmt(val, currency)` | Formats numbers as `$ X` (ARS) or `U$S X` (USD) |

## Adding real API data

Replace the `state.accounts` array with a fetch from the Meta Graph API. The shape each account needs:

```js
{
  id, name, accountId, currency,
  budget,  // monthly cap (editable)
  spent,   // month-to-date
  kpi,     // default KPI key: 'cpa' | 'roas' | 'ctr' | 'cpm' | 'cpl' | 'cpc'
  campaigns: [
    { name, status,        // 'active' | 'paused' | 'learning'
      spent, impressions, clicks, conversions, revenue }
  ]
}
```

Budget edits are in-memory only; persist them via `localStorage` or a backend call inside the `bi.addEventListener('change', ...)` handler in `render()`.

---
> Source: [kaptor-tech/copiloto-pauta](https://github.com/kaptor-tech/copiloto-pauta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
