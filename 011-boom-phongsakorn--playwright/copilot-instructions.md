## playwright

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Node.js Express REST API for a to-do list with Swagger documentation and Playwright E2E tests.

## Commands

```bash
# Start the server (port 3000)
npm start

# Run all Playwright tests
npx playwright test

# Run tests in headed mode (see browser)
npx playwright test --headed

# Run tests with Playwright Inspector debugger
npx playwright test --debug

# Run a single test file
npx playwright test tests/example.spec.js

# Run tests matching a pattern
npx playwright test -g "pattern"

# Run tests in a specific browser
npx playwright test --browser chromium

# Install Playwright browsers (required after fresh clone)
npx playwright install --with-deps
```

## Architecture

- **`src/app.js`** — Express server with all CRUD endpoints (`GET/POST/PUT/DELETE /todos`), Swagger/OpenAPI specs via JSDoc, and Swagger UI at `/api-docs`. Uses in-memory array storage with auto-incrementing IDs.
- **`tests/`** — Playwright E2E test specs (ES Modules). Currently contains example tests against external sites.
- **`playwright.config.js`** — Configures tests to run across Chromium, Firefox, and WebKit in parallel. CI mode uses 2 retries and a single worker. HTML reporter generates output to `playwright-report/`.

## Key Dependencies

- **express** + **cors** — API framework
- **swagger-jsdoc** + **swagger-ui-express** — API documentation
- **@playwright/test** — E2E testing

## CI

GitHub Actions (`.github/workflows/playwright.yml`) runs Playwright tests on push to main/master and on PRs. Uploads HTML report as an artifact.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/011-Boom-Phongsakorn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
