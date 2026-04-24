## trading

> This project is hosted on GitHub Pages at https://mon-ius.github.io/trading/

# Trading — Market Microstructure Simulator

## Deployment

This project is hosted on GitHub Pages at https://mon-ius.github.io/trading/

**After every code change, always commit and push to `master` so the live site stays current:**

```
git add <changed files>
git commit -m "description"
git push origin master
```

Never leave changes uncommitted or unpushed. The site must always reflect the latest code.

## Project structure

Static SPA — no build step. All files served directly:

- `index.html` — single page app (navbar, sidebar, chart/game views, slides, architecture, glossary)
- `css/style.css` — design tokens, dark mode, responsive layout
- `js/sim.js` — unified simulation engine: CDA, CARA agents, bubble metrics, experience replays, Trading Floor canvas
- `js/i18n.js` — EN + ZH translations
- `js/charts.js` — Plotly chart rendering (6 charts: price, volume, bubble, beliefs, P&L, experience)
- `js/ai-agent.js` — optional LLM provider hooks (Claude / GPT) for belief updates
- `js/app.js` — UI controller (theme, export, panel logic, event binding)
- `favicon.svg` — candlestick + alpha-star icon

## Key conventions

- i18n: every user-visible string uses `data-i18n` attributes + `t()` function
- Dark mode: CSS variables on `[data-theme]`, auto-detect via `prefers-color-scheme`
- Agent names: numbered format `"1.Ada"`, `"2.Ben"` — assigned in `runExperiment()`
- Risk sliders: linked comp-bar, always sum to 100% via `constrainRisk()`
- Charts: HTML `<h4 class="chart-title">` above Plotly div, Plotly has `margin.t: 8` (no internal title)
- Game canvas: separate mouse/touch/wheel handlers — never use pointer events (conflict-prone)
- Sidebar: unified panels (no mode tabs), market params and experience controls
- Export: JSON and CSV from `_result` (single experiment result with sessionResults array)

## Research context

Based on TWO papers:
- Dufwenberg, Lindqvist & Moore (2005, AER) "Bubbles and Experience" — declining FV asset, CDA market, experience effect
- Lopez-Lira (2025) "Can LLMs Trade?" — CARA utility agents, reservation prices, optional LLM substitution

The project replaces human subjects in the DLM experiment with Lopez-Lira CARA / LLM agents. Two experience levers: within-session α (DLM transfer treatment) and across-session experience replays (DLM main result, modelled as bias × 0.5^experience).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mon-ius) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
