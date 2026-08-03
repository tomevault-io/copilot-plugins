## charter

> This repository is an Electron desktop application. For rendered application UI testing and

# Repository Agent Instructions

## Electron UI validation

This repository is an Electron desktop application. For rendered application UI testing and
visual QA, use the repository's Playwright Electron workflow directly.

- Do not try the Codex in-app Browser for Charter UI validation unless the user explicitly asks
  for browser-based testing.
- Launch automated UI tests with the existing Electron helpers and isolated user-data directory
  in `tests/e2e/helpers/launch.ts`.
- Prefer targeted runs while iterating, for example:

  ```sh
  npx playwright test --config tests/e2e/playwright.config.ts tests/e2e/<target>.spec.ts
  ```

- Run `npm run build` before Electron E2E when renderer, preload, main-process, or shared package
  source has changed.
- Store temporary screenshots, traces, and ad hoc QA artifacts outside the repository, normally
  under `/tmp`. Do not commit generated QA artifacts unless the user explicitly requests them.
- For visual changes, validate the real Electron surface at the intended desktop viewport and a
  narrower viewport when practical. Check page identity, non-blank rendering, framework overlays,
  relevant console/page errors, screenshot evidence, and the primary interaction path.

---
> Source: [longyunfeigu/Charter](https://github.com/longyunfeigu/Charter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
