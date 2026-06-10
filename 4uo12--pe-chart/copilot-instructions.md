## pe-chart

> This repository is a local browser-based PE data analysis prototype. Runtime source files are grouped by role:

# Repository Guidelines

## Project Structure & Module Organization

This repository is a local browser-based PE data analysis prototype. Runtime source files are grouped by role:

- `pages/`: application pages (`index.html`, `access.html`, `analysis.html`, `delivery.html`).
- `src/app.js`: Excel/CSV parsing, state management, chart rendering, filters, and report export.
- `src/platform-adapters.js`: shared data model helpers, quality reporting, API adapter hooks.
- `assets/css/styles.css`: application layout and visual styling.
- `assets/images/`: page hero images.
- `server.js`: Node.js static server plus lightweight in-memory APIs.
- `docs/requirements/`: extracted and original product requirement material.

There is currently no dedicated `tests/` directory or build output directory.

## Build, Test, and Development Commands

Run locally from the repository root:

```powershell
node server.js
```

Starts the app at `http://127.0.0.1:8080/index.html` with compatibility routing to `pages/index.html`.

Use a different port when needed:

```powershell
$env:PORT=8787
node server.js
```

Quick service check:

```powershell
Invoke-WebRequest http://127.0.0.1:8080/api/health
```

No package installation or build command is required; the app loads XLSX and ECharts from CDN unless `pages/*.html` is changed to use local files.

## Coding Style & Naming Conventions

Use plain JavaScript, HTML, and CSS. Match existing style: two-space indentation, semicolons in JavaScript, `const`/`let` instead of `var`, and descriptive camelCase names such as `createQualityReport`, `studentStore`, and `renderRadar`.

Keep browser logic in `src/app.js`, reusable platform/data helpers in `src/platform-adapters.js`, page markup in `pages/`, styling in `assets/css/styles.css`, and server-only logic in `server.js`. Avoid broad refactors when making feature changes.

## Testing Guidelines

No automated test framework is configured. For changes, perform manual verification:

- Start `node server.js`.
- Open `/index.html` or `/pages/analysis.html`.
- Import user-provided local Excel/CSV files and confirm charts, filters, KPIs, quality report, and report export still work.
- Check `/api/health`, `/api/imports`, and `/api/reports` when touching server or adapter code.

If adding tests, document the framework and command in `README.md` and this file.

## Commit & Pull Request Guidelines

This directory is not currently a Git repository, so no existing commit convention is available. If Git is introduced, prefer concise imperative commits, for example `Add demo data import validation`.

Pull requests should include a short summary, affected files, manual test results, screenshots for UI changes, and notes for any data-format or API behavior changes.

## Security & Configuration Tips

Do not commit real student data or bundled example student datasets. `server.js` stores imports and reports in memory only; do not treat it as durable storage. Preserve static-file path checks when editing the server.

## Agent skills

### Issue tracker

Issues and PRDs are tracked as local markdown files under `.scratch/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Triage uses Chinese local-markdown status labels mapped from the canonical skill roles. See `docs/agents/triage-labels.md`.

### Domain docs

This repo uses a single-context domain documentation layout. See `docs/agents/domain.md`.

---
> Source: [4uo12/PE-chart](https://github.com/4uo12/PE-chart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
