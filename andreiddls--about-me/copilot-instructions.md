## about-me

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```
npm run dev        # Start Vite dev server (http://localhost:5173)
```

## Architecture

This is a portfolio/CV site built as a **multi-page static site** with Vite + React (used only for chart widgets). No router — each page is a separate `.html` file.

### Data flow

All content lives in JSON files under `data/` — one JSON file per page. HTML pages contain empty containers with IDs; JS renders content into them at DOMContentLoaded.

```
data/cv-data.json        →  index.html  (main portfolio)
data/sc-platform-data.json →  sc_platform.html  (case study)
```

**To change content**, edit the JSON files directly. No need to touch HTML or JS.

### Key files

| File | Role |
|---|---|
| `js/data.js` | Re-exports `cvData` from `data/cv-data.json` |
| `js/main.js` | All DOM rendering: `renderAbout()`, `renderStats()`, `renderExperience()`, `renderSkills()`, `renderProjects()`, `renderEducation()`, `renderLanguages()`. Also exports `renderCaseStudy()`, `renderProjectInfo()`, `renderKeyAchievements()` for the case study page. |
| `js/charts.jsx` | React visx bar chart (`Before`/`After` response time). Exports `renderChart(containerId, data)`. Used in the Sberbank tile on index.html. |
| `js/FinalScoreCharts.jsx` | React visx charts for case study final scores (NPS and MAU). Exports `renderFinalScores(npsId, mauId, config)`. |
| `css/styles.css` | All styles — CSS custom properties for theming (light/dark), bento grid layout, glass panels, tiles. |

### Page structure

Both pages use a two-column bento grid layout (70/30 on desktop, single column on mobile):

- **`.col-left`** — main content tiles (about, projects, experience, case study sections)
- **`.col-right`** — sidebar glass panels (education, skills, project info, achievements)

Theming is handled via CSS custom properties on `:root`, toggled by a `theme-dark`/`theme-light` class on `<html>`. The theme toggle button is duplicated with inline JS in each HTML file.

### Dependencies

- **React 18** + **visx** — used only for SVG bar charts rendered into specific DOM containers
- **Vite** with `@vitejs/plugin-react` — handles JSX transforms and dev server

---
> Source: [andreiddls/about-me](https://github.com/andreiddls/about-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
