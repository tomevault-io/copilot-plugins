## x-test-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`@netflix/x-test-cli` is a Node.js CLI tool that automates browser testing for the `@netflix/x-test` browser-side test runner. It launches browsers (currently Puppeteer/Chrome), runs tests, collects coverage, and outputs TAP-compliant results.

## Architecture

The codebase consists of three main modules:

- **x-test-cli.js**: Entry point that parses CLI arguments and dispatches to the appropriate client
- **x-test-cli-puppeteer.js** (`XTestPuppeteerCli`): Puppeteer-specific implementation that launches browser, collects coverage, and validates TAP output
- **x-test-cli-common.js** (`XTestCliCommon`): Browser-side functions injected via `page.evaluate()` that communicate with the test runner using BroadcastChannel API

### Communication Flow

1. CLI launches browser and navigates to test URL
2. Browser-injected functions (`XTestCliCommon.run()` and `XTestCliCommon.cover()`) communicate with the test runner via BroadcastChannel named 'x-test'
3. Test runner outputs TAP to console, which CLI captures and validates
4. Coverage data flows: Browser → CLI → Back to browser for processing

### URL Parameters

- `x-test-name`: Test name filter (regex pattern), automatically added when `--test-name` CLI arg is provided
- `x-test-run-coverage`: Flag to enable coverage mode, automatically added when `--coverage=true`

## Development Commands

**Linting:**
```bash
npm run lint          # Check for lint errors (max 0 warnings)
npm run lint-fix      # Auto-fix lint errors
```

**Version Bumping:**
```bash
npm run bump          # Display current version
npm run bump 1.0.0    # Bump to specific version (also supports major/minor/patch)
```

**Testing the CLI:**
```bash
node x-test-cli.js --client=puppeteer --url=http://localhost:8080/test/ --coverage=true
```

## Publishing

Uses GitHub Actions to publish from tags. Two workflows:

1. **Manual**: Edit `package.json`, `jsr.json` versions → commit → tag → push → create GitHub release
2. **Assisted**: Run `npm run bump <version>` (handles all file updates and git operations) → push with `git push origin main --follow-tags` → create GitHub release

The bump script automatically updates `package.json`, `package-lock.json`, and `jsr.json`, then commits and tags.

## Code Conventions

- ES modules (`"type": "module"` in package.json)
- Kebab-case CLI args are converted to camelCase internally (e.g., `--test-name` → `testName`)
- TAP output flows through stdout; errors through stderr
- Exit code 1 on test failures or validation errors
- All publishable files listed in `package.json` "files" array

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Netflix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
