## code-to-uml

> Code-To-UML is a zero-dependency static frontend with a lightweight Node server.

# Repository Guidelines

## Project Structure & Module Organization

Code-To-UML is a zero-dependency static frontend with a lightweight Node server.

- `index.html`, `demo.html`, `demo.js`, `main.css`: main browser UI, demo runtime, and global styling.
- `serve.js`, `serve.sh`, `serve.bat`: local static server and PlantUML rendering fallback.
- `component/`: reusable browser modules such as TOC, report cards, sorting, and failure handling.
- `data/`: `.ctu` report/example data; `data/_TEMPLATE.ctu` defines the data format.
- `cache/`: generated HTML reports and `cache/_TEMPLATE.html`.
- `i18n/` and `i18n-config.js`: English/Chinese localization.
- `js/`: bundled third-party PlantUML and diagram libraries; avoid editing minified vendor files.
- `test/`: Node-based unit/static tests, run through `test/run-all.js`.
- `skills/`: AI-agent skill definitions and validators for Code-To-UML report generation.

## Build, Test, and Development Commands

- `npm start` or `node serve.js`: start the local server on the default port.
- `npm run serve` or `node serve.js 5401`: start the server on port `5401`.
- `./serve.sh 5401` / `serve.bat 5401`: platform launchers for local development.
- `npm test`: run all tests via `node test/run-all.js`.
- `npm run lint`: syntax-check core JavaScript files with `node --check`.
- `npm run prepublishOnly`: run lint and tests before publishing.

Node.js 18+ is required. Java is required only for server-side PlantUML rendering through `plantuml.jar`.

## Coding Style & Naming Conventions

Use vanilla ES6+ JavaScript with `"use strict"` and semicolons. Follow existing indentation and formatting in the touched file. Prefer `camelCase` for variables/functions and `UPPER_SNAKE_CASE` for constants. Keep browser logic in `demo.js` or `component/`; do not modify `js/*.min.js` vendor assets. Use semantic HTML, `data-*` hooks for runtime behavior, and CSS custom properties for themeable styles.

## Testing Guidelines

Tests are plain Node scripts under `test/`, generally named `<feature>.test.js` or `<feature>-static.test.js`. Add focused tests for changed parser, server, template, rendering, install, or static-contract behavior. Run `npm test` before submitting changes; run `npm run lint` when JavaScript files change. For visual/report changes, also start the server and verify `http://localhost:5401`.

## Commit & Pull Request Guidelines

Use short conventional-style commits seen in history, for example `feat(template): enhance page introduction` or `docs(readme): add demo image sections`. Keep related changes together.

Pull requests should include a clear description, linked issue when relevant, change type, test steps, and screenshots for UI/report visual changes. Confirm local server testing, diagram rendering, and mobile responsiveness when applicable.

---
> Source: [pingwurth/code-to-uml](https://github.com/pingwurth/code-to-uml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
