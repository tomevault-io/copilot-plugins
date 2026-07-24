## jest-junit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

jest-junit is a Jest reporter that generates JUnit-compatible XML test result files. It supports both the modern reporter API and the deprecated testResultsProcessor API.

## Commands

- **Run tests**: `NODE_OPTIONS=--experimental-vm-modules npm test`
- **Run tests and update snapshots**: `NODE_OPTIONS=--experimental-vm-modules npm test -u`
- **Run a single test file**: `NODE_OPTIONS=--experimental-vm-modules npx jest __tests__/getOptions.test.js`
- **Run tests matching a pattern**: `NODE_OPTIONS=--experimental-vm-modules npx jest -t "should generate valid xml"`

The `--experimental-vm-modules` flag is required for Jest 30+.
Always run `npm install` if any changes are made to package.json before running tests or committing.

## Architecture

### Entry Point

`index.js` - Exports the `JestJUnit` class that serves as both:
- A Jest reporter (when instantiated with `new`)
- A testResultsProcessor (deprecated, for legacy support)

The reporter collects console output via `onTestResult` and generates XML on `onRunComplete`.

### Core Modules

- `utils/buildJsonResults.js` - Transforms Jest's test results into the XML-compatible JSON structure consumed by the `xml` package. Handles template variable replacement for suite/class/title names.
- `utils/getOptions.js` - Merges configuration from three sources (in order of precedence): environment variables > package.json `jest-junit` key > reporter options > defaults
- `utils/getOutputPath.js` - Resolves the output file path, handling `<rootDir>` replacement
- `constants/index.js` - Default options and environment variable mappings

### Configuration Flow

Options are resolved in `getOptions.js` with this precedence:
1. Environment variables (e.g., `JEST_JUNIT_OUTPUT_DIR`)
2. `jest-junit` key in package.json
3. Reporter options passed in jest.config.js
4. Default values from `constants/index.js`

### Template Variables

Templates (`suiteNameTemplate`, `classNameTemplate`, `titleTemplate`) support these placeholders:
- `{filepath}`, `{filename}`, `{title}`, `{classname}`, `{suitename}`, `{displayName}`

Templates can also be functions (reporter options only) receiving a variables object.

### Custom Properties

Users can add custom properties to test suites/cases via:
- `junitProperties.js` - Suite-level properties
- `junitTestCaseProperties.js` - Test case-level properties

---
> Source: [jest-community/jest-junit](https://github.com/jest-community/jest-junit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
