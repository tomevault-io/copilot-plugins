## d3-lwc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Salesforce LWC library providing 10 D3.js chart components for use in Lightning App Builder, Flows, and Experience Builder. Each chart accepts data via `recordCollection` (from Flow/parent) or `soqlQuery` (Apex-backed SOQL).

## Commands

```bash
npm test                                        # Run all unit tests
npm test -- --testPathPattern=d3BarChart         # Run tests for a specific component
npm run test:unit:watch                         # Watch mode
npm run test:unit:coverage                      # With coverage report
npm run lint                                    # ESLint
npm run prettier                                # Format all files
npm run prettier:verify                         # Check formatting

# Deploy to Salesforce org
sf project deploy start --source-dir force-app -o <org-alias>

# Start local dev server (use Node 20, not 25)
export PATH="/opt/homebrew/opt/node@20/bin:$PATH"
sf lightning dev app -o <org-alias>
```

Pre-commit hook (husky + lint-staged) auto-runs Prettier, ESLint, and related Jest tests on staged files.

## Architecture

### Data Flow

1. **Server-preferred path** (when `objectApiName` + field config available):
   - Aggregation charts (bar, donut, treemap): `getAggregatedData` → server-side GROUP BY → pre-bucketed results
   - Histogram statistics: `getStatistics` → server-side mean/median/stdDev (raw values still fetched for binning)
   - Scatter correlation: `getCorrelation` → server-side Pearson r, slope, intercept (raw points still fetched for rendering)

2. **Client-side path** (recordCollection or soqlQuery-only fallback):
   - `recordCollection` or `soqlQuery` → Apex `D3ChartController.executeQuery()`
   - `dataService.validateData()` → `validateFields()` → `truncateData()` (2,000 record limit) → `aggregateData()`
   - Processed data → D3 renders into an empty `<div>`

### Shared Modules

- `d3Lib` — D3.js loader with singleton pattern and fetch+eval fallback for CSP-restricted environments. Use `loadD3(this)` first call, `getD3()` after, `resetD3()` in tests.
- `dataService` — Data validation, aggregation (Sum/Count/Average), truncation (MAX_RECORDS: 2,000)
- `themeService` — 4 color palettes (Salesforce Standard, Warm, Cool, Vibrant) + custom colors
- `chartUtils` — Number formatting, tooltips, resize handling, layout retry

### Chart Component Pattern

Every chart component follows this structure:

- **@api properties**: `recordCollection`, `soqlQuery`, field mappings (`groupByField`/`valueField` or `xField`/`yField`), `operation`, `height`, `theme`, `advancedConfig` (JSON string), `objectApiName`/`filterField` for drill-down
- **Lifecycle**: `connectedCallback` loads D3 + fetches data; `renderedCallback` initializes chart with layout retry for container measurement; `disconnectedCallback` cleans up ResizeObserver
- **State guards**: `chartRendered` flag prevents re-rendering; `_layoutRetry` handles cases where container has no dimensions yet

### Apex Controller

`D3ChartController` (`with sharing`) — four `@AuraEnabled(cacheable=true)` methods:

- `executeQuery(queryString)` — Raw SOQL execution with FLS enforcement. Auto-adds LIMIT 2000.
- `getAggregatedData(objectName, groupByField, valueField, operation, filterClause)` — Server-side GROUP BY aggregation. Validates object/field existence via Schema describe. Returns label/value pairs. LIMIT 200 groups.
- `getStatistics(queryString, valueField)` — Computes count, min, max, mean, median, and population stdDev server-side.
- `getCorrelation(queryString, xField, yField)` — Computes Pearson correlation coefficient, linear regression slope and intercept.

### Testing

- Mocks in `__mocks__/` for `lightning/platformResourceLoader`, `lightning/navigation`, `lightning/platformShowToastEvent`, and `@salesforce/apex/D3ChartController.executeQuery`
- Tests create a mock D3 factory with chainable method stubs (since D3 isn't available in jsdom)
- Jest config extends `@salesforce/sfdx-lwc-jest/config` with custom `moduleNameMapper` for the mocks above

### Conventions

- Component names prefixed with `d3` (e.g., `d3BarChart`)
- `// ABOUTME:` comments at top of component files for component-level documentation
- Constants use UPPER_SNAKE_CASE (`MAX_RECORDS`, `OPERATIONS`, `PALETTES`)
- HTML templates use SLDS classes with conditional rendering for loading/error/no-data/chart states

## Chart Backlog

- **ROADMAP.md** — Next 16 charts (Weeks 1–16), detailed specs with Salesforce use cases
- **CHART-INDEX.md** — Next 50 charts beyond the roadmap, ordered by complexity (1 = simplest → 50 = most complex). When deciding what to build next, consult this index and build in order. Each entry includes: D3 gallery reference, proposed component name, Salesforce use case, and new D3 concepts required.

Total library target: 76 charts (10 built + 16 roadmap + 50 index).

## Key Constraints

- Node.js v20 required for Salesforce CLI compatibility (v25 has issues)
- D3.js v7 loaded from `staticresources/d3.js`
- Salesforce API version: 65.0

## Two-Project Sync Strategy

This project is the **source of truth**. Changes flow one direction: d3-lwc → agentforce-dev.

### Path Mapping

| d3-lwc                            | agentforce-dev                            |
| --------------------------------- | ----------------------------------------- |
| `force-app/main/default/lwc/`     | `force-app/main/d3/lwc/`                  |
| `force-app/main/default/classes/` | `force-app/main/d3/classes/`              |
| `__mocks__/`                      | `__mocks__/` (project root)               |
| `jest.config.js`                  | `jest.config.js` (merge moduleNameMapper) |

### Sync Checklist

After making changes here, sync to `~/code/agentforce-dev/`:

1. **Apex classes** — full replace (cls + meta.xml)
2. **Service JS** (dataService, chartUtils, themeService, d3Lib) — full replace
3. **Chart component JS** — full replace
4. **Test files** — ALL tiers: unit (.test.js), integration (.integration.test.js), e2e (.e2e.test.js)
5. **Mock files** (`__mocks__/`) — full replace
6. **jest.config.js** — merge moduleNameMapper (don't replace — agentforce-dev has other config)
7. **meta.xml** — MERGE, don't replace. agentforce-dev has `lightningCommunity__Page` targets to preserve.

### Automation

Run `scripts/sync-to-agentforce.sh` to automate steps 1-5. Steps 6-7 require manual review.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/weytani)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/weytani)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
