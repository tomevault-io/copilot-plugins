## reseller-research

> This is a dependency-free static web prototype. The root `index.html` is the browser entry point and loads all application code from `src/`.

# Repository Guidelines

## Project Structure & Module Organization

This is a dependency-free static web prototype. The root `index.html` is the browser entry point and loads all application code from `src/`.

- `index.html`: page markup and script/style references.
- `src/app.js`: application logic, mock eBay pricing data, mock AI insights, rendering, and PDF print handoff.
- `src/styles.css`: layout, visual styling, responsive rules, and print styles.
- `README.md`: project overview, current flow, and future integration notes.

There is currently no dedicated `tests/` directory or asset folder. Add new source files under `src/` and keep paths relative to `index.html`.

## Build, Test, and Development Commands

No package manager or build step is required.

- Open `index.html` in a browser to run the app locally.
- Use the browser refresh button after editing `src/app.js` or `src/styles.css`.
- Use **Download PDF** in the UI to validate the print/PDF flow.

If a future toolchain is added, document the exact commands here and keep `README.md` in sync.

## Coding Style & Naming Conventions

Use plain JavaScript, HTML, and CSS. Keep indentation at two spaces for HTML and JavaScript, matching the current files. Prefer descriptive camelCase names for JavaScript functions and variables, such as `generateMockAiInsights()` and `fetchMockEbayAverageSellingPrice()`. Use kebab-case for CSS classes and IDs, such as `brand-form`, `sheet-panel`, and `download-pdf-button`.

Keep mock-data shapes stable where possible: `brand`, `generatedAt`, `source`, `categories[]`, and `aiInsights`.

## Testing Guidelines

There is no automated test framework yet. For changes, manually verify:

- The default Patagonia sheet generates successfully.
- A custom brand input renders a complete report.
- Loading and disabled button states behave correctly.
- The PDF print flow opens and uses print-specific styling.
- The layout remains usable on narrow mobile widths.

If tests are introduced, use focused browser or DOM tests and name files after the behavior under test, for example `app-rendering.test.js`.

## Commit & Pull Request Guidelines

This repository has no commit history yet, so use concise imperative commit messages, for example `Add brand report export state` or `Refine mobile sheet layout`.

Pull requests should include a short summary, the manual verification performed, screenshots for UI changes, and any notes about changed mock data or future API integration points.

## Security & Configuration Tips

Do not commit API keys or reseller account credentials. When replacing mock eBay or AI functions with real services, route secrets through a backend or environment-based configuration rather than browser JavaScript.

---
> Source: [jjh044/Reseller-Research](https://github.com/jjh044/Reseller-Research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
