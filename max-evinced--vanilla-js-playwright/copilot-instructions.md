## vanilla-js-playwright

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Playwright test framework integrated with the Evinced SDK for automated accessibility testing. It demonstrates various patterns for integrating Evinced's accessibility scanning capabilities into end-to-end tests using vanilla JavaScript.

## Environment Setup

The project requires Evinced SDK credentials to be set as environment variables:
- `EVINCED_SERVICE_ID`: Your Evinced service identifier
- `EVINCED_API_KEY`: Your Evinced API secret key

These are configured in `global-evinced-setup.js` which runs before all tests via the `globalSetup` configuration in `playwright.config.js`.

## Running Tests

```bash
# Run main Evinced test suite (tests/evinced directory)
npm test

# Run ticket-specific tests (tests/tickets directory)
npm run tickets

# Run a single test file
npx playwright test tests/path/to/file.spec.js

# Run tests in UI mode
npx playwright test --ui

# Run tests in debug mode
npx playwright test --debug

# Run tests in headed mode (visible browser)
npx playwright test --headed tests/path/to/file.spec.js

# Repeat a test multiple times (requires zsh)
repeat 5 { npx playwright test --headed ./tests/path/to/some.spec.js }
```

## Test Architecture

The codebase demonstrates two main patterns for Evinced integration:

### 1. Direct SDK Usage Pattern
Used in `tests/evinced/evinced-global.spec.js` and most `tests/tickets/` files:
```javascript
const { EvincedSDK } = require('@evinced/js-playwright-sdk');

test('test name', async ({ page }) => {
  await page.goto('https://example.com');
  const evincedService = new EvincedSDK(page);
  const issues = await evincedService.evAnalyze();
  evincedService.evSaveFile(issues, 'html', 'test-results/report.html');
});
```

### 2. Fixture Pattern (Recommended for Continuous Mode)
Used in `tests/evinced/simple-fixture.spec.js`:
```javascript
import { test, expect } from '../../fixtures/evincedFixture.js'

test('test name', async ({ page, evincedContMode }) => {
  await page.goto('https://example.com');
  // evincedContMode fixture automatically:
  // - Calls evStart() before the test
  // - Calls evStop() and evSaveFile() after the test
});
```

The `evincedFixture.js` automatically handles starting continuous scanning (`evStart()`), stopping it (`evStop()`), and saving reports to `test-results/` with the test name as the filename.

### 3. Component Testing
The `tests/feature/component-testing-gen1.spec.js` demonstrates advanced component-specific analysis:
- Uses `evinced.components.analyzeButton()` and `evinced.components.analyzeCombobox()`
- Merges component-specific issues with continuous mode results using `evMergeIssues()`

## Test Organization

- **`tests/evinced/`**: Core Evinced integration examples demonstrating basic usage patterns
- **`tests/tickets/`**: Bug reproductions and customer-specific scenarios (e.g., performance testing, iframe handling, WCAG filtering)
- **`tests/feature/`**: Feature-specific tests for new SDK capabilities
- **`fixtures/evincedFixture.js`**: Reusable Playwright fixture that automates Evinced continuous mode workflow

## Key Configuration Details

- **Browser Support**: Currently configured to run on Chromium only (playwright.config.js:39-42). Firefox and WebKit are commented out due to an NSBINDING issue.
- **Parallelization**: Tests run in parallel by default (`fullyParallel: true`)
- **Reporter**: Uses Evinced's custom reporter at `node_modules/@evinced/js-playwright-sdk/dist/reporter/evincedReporter.js` alongside HTML and line reporters
- **Test Results**: All Evinced reports are saved to `test-results/` directory

## Common Evinced SDK Methods

- `evAnalyze(options?)`: Run a one-time accessibility scan
- `evStart(initOptions?)`: Start continuous monitoring mode
- `evStop(options?)`: Stop continuous monitoring and return collected issues
- `evSaveFile(issues, format, destination)`: Save results in one of 4 formats: `'json'`, `'html'`, `'sarif'`, or `'csv'`
- `components.analyzeButton(selector)`: Analyze a specific button component
- `components.analyzeCombobox(selector)`: Analyze a specific combobox component
- `evMergeIssues(...listsOfIssues)`: Combine multiple issue sets into a single deduplicated list

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/max-evinced)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/max-evinced)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
