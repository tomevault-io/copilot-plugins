## wcag-audit

> > Generated: 2026-02-26

# WCAG Audit CLI — Redesign Plan

> Generated: 2026-02-26
> Completed: 2026-02-27
> Status: **All Phases Complete** ✅

---

## Overview

Restructure the monolithic 3-script WCAG audit pipeline (~2,635 lines across 3 files) into a modular Commander.js CLI app with subcommands (`wcag-audit audit`, `wcag-audit report`, `wcag-audit extract-routes`). The existing code moves untouched to `rough_draft/`. The new structure splits code across ~20 focused modules organized by responsibility.

Work proceeds in 6 phases. Each phase ends with an end-to-end verification step and a commit point.

### Key Decisions

| Decision | Choice | Rationale |
|---|---|---|
| CLI framework | Commander.js | Most widely used Node CLI framework, zero build step, mature ESM support |
| File extensions | `.js` with `"type": "module"` | Cleaner than `.mjs`, entire repo is ESM |
| Entry pattern | Subcommands | Single entry point, self-documenting `--help`, consistent option parsing |
| False-positive toggle | `createFilter(enabled)` pattern | Returns real or no-op functions — no `if` checks scattered through engine code |
| Third-party disclaimers | From existing JSONC rule map | `thirdPartyVendors` already has all needed fields |
| Slug style | Clean dashes | `/invoice/:id/payment` → `invoice-id-payment` (no `dyn-` prefix, no `__`) |
| iFrame scanning | Opt-in via `--scan-iframes` | Default false; no explicit Stripe exclusion needed |
| Route extraction | Standalone subcommand | Not part of audit pipeline — run separately or as CI pre-step |
| CSS management | External `.css` file, inlined at generation time | Report stays self-contained; CSS becomes editable |
| Test strategy | Manual verification at each phase | CLI smoke tests and output comparison against rough draft |

### Architectural Principles

These apply across every module:

1. **Thin commands, fat services.** Command handlers are ~20-line adapters that parse CLI input, build a context, and delegate. All domain logic lives in service modules.

2. **Session context object.** A single `AuditContext` / `ReportContext` object is created by the command handler and threaded through the pipeline. Cross-cutting concerns (logger, output dir, timeouts, filter, browser instances) live on this object. Adding a concern means changing one object, not every function signature.

3. **Structured logger.** No bare `console.log`. A `createLogger({ quiet, verbose })` function returns `{ info, debug, warn, error, success }` methods. Lives on the context. Enables `--quiet` for CI and `--verbose` for debugging.

4. **No side effects at module scope.** Every module exports pure functions (or functions that accept a context). No `fs.mkdirSync`, no `setConfig()`, no `throw` at the top level. All side effects happen in command handlers. You can `import` any module without triggering I/O.

5. **Consistent error strategy.** Commands catch at the top and set `process.exitCode = 1`. Everything else throws. Commander's `.exitOverride()` keeps this clean.

6. **Barrel exports.** Each `src/` subdirectory has an `index.js` re-exporting its public API. Consumers write `import { normalizeSlug, esc } from '../utils/index.js'` instead of three separate imports. Internal refactoring stays safe.

7. **Resolve assets via `import.meta.url`.** `styles.css` and other file-relative assets use `new URL('./styles.css', import.meta.url)` — works regardless of `process.cwd()`.

8. **Output writer abstraction.** A shared `writeOutputs(outputDir, manifest, log)` function handles file writing + logging. One place to add dry-run mode, compression, or S3 upload later.

9. **Split defaults by domain.** Audit-time constants (`TIMEOUTS`, `VIEWPORT`) and report-time constants (`LIMITS`, fallback rule maps) live in separate files so the two commands don't cross-load unused config.

10. **Type safety without TypeScript.** A `jsconfig.json` enables VS Code's `checkJs` + IntelliSense from JSDoc annotations. Zero build step, full autocomplete and type checking.

---

## Directory Structure

```
wcag-audit/
├── rough_draft/                    # Backup of all current files (DO NOT DELETE)
│   ├── run-wcag-audit.mjs
│   ├── generate-wcag-conformance-report.mjs
│   ├── extract-routes.mjs
│   ├── run-wcag-pipeline.sh
│   └── package.json.bak
│
├── bin/
│   └── wcag-audit.js               # Commander entry point (~40 lines)
│
├── src/
│   ├── commands/
│   │   ├── audit.js                # ~25 lines: parse opts → build context → delegate
│   │   ├── report.js               # ~25 lines: parse opts → build context → delegate
│   │   └── extract-routes.js       # ~20 lines: parse opts → delegate to extractors
│   │
│   ├── engines/
│   │   ├── ibm.js                  # configureIbm(), runIbmScan(ctx), closeIbm()
│   │   ├── axe.js                  # runAxeScan(ctx)
│   │   ├── lighthouse.js           # runLighthouseScan(ctx), normalizeLighthouseViolations()
│   │   └── index.js                # barrel: re-exports all engines
│   │
│   ├── filters/
│   │   ├── false-positives.js      # createFilter(enabled) + individual filter fns
│   │   └── index.js                # barrel
│   │
│   ├── scanner/
│   │   ├── route-scanner.js        # scanRoutes(ctx) — main loop
│   │   ├── interactive-states.js   # INTERACTIVE_STATES array
│   │   ├── iframe-scanner.js       # getIframeTargets(), scanIframesForPage(ctx)
│   │   └── index.js                # barrel
│   │
│   ├── server/
│   │   ├── lifecycle.js            # isLocalUrl(), ensureAppInstance(ctx), stopManagedApp()
│   │   └── index.js                # barrel
│   │
│   ├── extractors/
│   │   ├── detect.js               # detectProjectType()
│   │   ├── expo.js                 # extractExpoRoutes()
│   │   ├── vite.js                 # extractViteRoutes()
│   │   ├── react-router.js         # extractReactRouterFsRoutes()
│   │   ├── navigation.js           # extractNavigationRoutes()
│   │   └── index.js                # barrel
│   │
│   ├── report/
│   │   ├── builder.js              # buildReport(), classifyThirdParty(), detectPatterns()
│   │   ├── html-generator.js       # generateHtml() + generateThirdPartyHtml() + disclaimers
│   │   ├── csv-generator.js        # generateCsv()
│   │   ├── linear-export.js        # buildLinearExports(), Jira CSV
│   │   ├── styles.css              # Extracted CSS (loaded via import.meta.url)
│   │   └── index.js                # barrel
│   │
│   ├── config/
│   │   ├── audit-defaults.js       # TIMEOUTS, VIEWPORT
│   │   ├── report-defaults.js      # LIMITS, LIGHTHOUSE_MANUAL_RULES, IBM/AXE_FALLBACK_RULES
│   │   ├── loader.js               # loadConfig(ruleMapPath), validateRuleMapConfig()
│   │   └── index.js                # barrel
│   │
│   ├── constants/
│   │   ├── wcag.js                 # SC_TITLES, PRINCIPLE_MAP, getCriterionMeta()
│   │   └── index.js                # barrel
│   │
│   └── utils/
│       ├── slugs.js                # normalizeSlug(), ibmReportSlug(), ibmReportPath()
│       ├── paths.js                # resolveOutputDir(), resolveMaybeRelative()
│       ├── browser.js              # openRoutePage(), waitForSettled(), clickFirst(), etc.
│       ├── text.js                 # esc(), escapeCsv(), severity helpers
│       ├── logger.js               # createLogger({ quiet, verbose })
│       ├── output.js               # writeOutputs(outputDir, manifest, log)
│       └── index.js                # barrel
│
├── jsconfig.json                   # checkJs, ES2022 module, node16 resolution
├── package.json                    # "type": "module", "bin" field
├── wcag-rule-map.example.jsonc
├── README.md
├── WCAG_progress.md
└── Claude.md                       # This file
```

### Module Responsibilities

| Module | Lines (est.) | Source | What moves here |
|---|---|---|---|
| `bin/wcag-audit.js` | ~40 | New | Commander program, 3 subcommand registrations |
| `src/commands/audit.js` | ~25 | `run-wcag-audit.mjs` | Build `AuditContext`, delegate to `scanRoutes()` + lifecycle |
| `src/commands/report.js` | ~25 | `generate-wcag-conformance-report.mjs` | Build `ReportContext`, delegate to builder + generators |
| `src/commands/extract-routes.js` | ~20 | `extract-routes.mjs` | Resolve opts, delegate to extractors |
| `src/engines/ibm.js` | ~60 | `run-wcag-audit.mjs` L57-77, L404-435 | `configureIbm()`, `runIbmScan(ctx)`, `closeIbm()` |
| `src/engines/axe.js` | ~30 | `run-wcag-audit.mjs` L437-455 | `runAxeScan(ctx)` |
| `src/engines/lighthouse.js` | ~60 | `run-wcag-audit.mjs` L372-402, L457-467 | `runLighthouseScan(ctx)`, `normalizeLighthouseViolations()` |
| `src/filters/false-positives.js` | ~120 | `run-wcag-audit.mjs` L79-175 | All FP logic + `createFilter(enabled)` toggle |
| `src/scanner/route-scanner.js` | ~150 | `run-wcag-audit.mjs` L667-820 | `scanRoutes(ctx)` main loop, `runAllEngines(ctx)` |
| `src/scanner/interactive-states.js` | ~25 | `run-wcag-audit.mjs` L177-202 | `INTERACTIVE_STATES` array |
| `src/scanner/iframe-scanner.js` | ~90 | `run-wcag-audit.mjs` L521-665 | `getIframeTargets()`, `scanIframesForPage(ctx)` |
| `src/server/lifecycle.js` | ~150 | `run-wcag-audit.mjs` L204-355 | App start/stop, port detection, health checks |
| `src/extractors/*.js` | ~260 | `extract-routes.mjs` L56-280 | Route extraction (one file per strategy) |
| `src/report/builder.js` | ~250 | `generate-wcag-conformance-report.mjs` L340-775 | Finding extraction, classification, pattern detection |
| `src/report/html-generator.js` | ~300 | `generate-wcag-conformance-report.mjs` L820-1170 | `generateHtml()`, third-party HTML, disclaimers |
| `src/report/csv-generator.js` | ~40 | `generate-wcag-conformance-report.mjs` L780-815 | `generateCsv()` |
| `src/report/linear-export.js` | ~250 | `generate-wcag-conformance-report.mjs` L1175-1420 | Linear/Jira export logic |
| `src/report/styles.css` | ~70 | `generate-wcag-conformance-report.mjs` ~L997-1070 | Extracted CSS |
| `src/config/loader.js` | ~110 | `generate-wcag-conformance-report.mjs` L104-210 | Config loading + JSONC parsing + validation |
| `src/config/audit-defaults.js` | ~20 | `run-wcag-audit.mjs` L44-55 | TIMEOUTS, VIEWPORT |
| `src/config/report-defaults.js` | ~60 | `generate-wcag-conformance-report.mjs` L45-102 | LIMITS, fallback rule maps |
| `src/constants/wcag.js` | ~70 | `generate-wcag-conformance-report.mjs` L220-285 | SC titles, principles, criterion metadata |
| `src/utils/slugs.js` | ~40 | Both scripts (deduplicated) | Slug normalization, IBM report paths |
| `src/utils/paths.js` | ~20 | Both scripts (deduplicated) | Path resolution helpers |
| `src/utils/browser.js` | ~80 | `run-wcag-audit.mjs` L367-519 | Puppeteer page helpers |
| `src/utils/text.js` | ~40 | `generate-wcag-conformance-report.mjs` L310-338 | HTML/CSV escaping, severity helpers |
| `src/utils/logger.js` | ~25 | New | `createLogger({ quiet, verbose })` |
| `src/utils/output.js` | ~15 | New | `writeOutputs(outputDir, manifest, log)` |

---

## Context Objects

### AuditContext

Created once by `src/commands/audit.js`, passed to all audit-time functions:

```js
/** @typedef {object} AuditContext
 * @property {import('puppeteer').Browser} browser
 * @property {import('chrome-launcher').LaunchedChrome} chrome
 * @property {{ filterIbm, filterAxe, patchIbmHtml }} filter
 * @property {string} baseUrl
 * @property {string} outputDir
 * @property {string} ibmOutputDir
 * @property {{ appReachable: number, appReady: number, ... }} timeouts
 * @property {{ width: number, height: number }} viewport
 * @property {boolean} scanIframes
 * @property {ReturnType<typeof createLogger>} log
 */
```

All engine runners, scanners, and browser helpers receive `ctx` as their first (or only) parameter. Adding a concern (like a progress callback or metrics collector) means adding one field — zero function signature changes downstream.

### ReportContext

Created once by `src/commands/report.js`:

```js
/** @typedef {object} ReportContext
 * @property {object} config       — merged config from JSONC + defaults
 * @property {string} outputDir
 * @property {string} inputFile
 * @property {ReturnType<typeof createLogger>} log
 */
```

---

## CLI Interface

### `wcag-audit audit`

Scan routes for WCAG violations using IBM Equal Access, axe-core, and Lighthouse.

```
Options:
  --routes-file <path>    Path to routes JSON file
  --route <route>         Single route to audit (e.g. /welcome)
  --base-url <url>        Base URL to scan (default: $AUDIT_BASE_URL or http://localhost:8081)
  --output-dir <dir>      Output directory (default: $WCAG_OUTPUT_DIR or ./audit)
  --start-cmd <cmd>       Command to start the app (default: $AUDIT_START_CMD or "yarn web:ci")
  --project-root <dir>    Project root (default: $WCAG_PROJECT_ROOT or cwd)
  --scan-iframes          Scan iframes on each page (default: false)
  --no-filter-fp          Disable false-positive filtering
  --quiet                 Suppress progress output (CI-friendly)
  --verbose               Show debug-level output
```

**Route resolution logic:**
- `--route /billing` → scan only `/billing` (no routes file required)
- `--routes-file routes.json` → scan all routes in the file
- `--routes-file routes.json --route /billing` → filter file to routes matching `/billing`
- Neither provided → look for `routes.json` in project root, error if not found

**Environment variable fallbacks:** Every CLI option falls back to its corresponding env var (`AUDIT_BASE_URL`, `AUDIT_START_CMD`, `AUDIT_SCAN_IFRAMES`, `AUDIT_ROUTE`, `WCAG_ROUTES_FILE`, `WCAG_OUTPUT_DIR`, `WCAG_PROJECT_ROOT`). CLI options take precedence.

### `wcag-audit report`

Generate conformance reports from audit output.

```
Options:
  --input <path>          Path to consolidated.json (default: <output-dir>/consolidated.json)
  --output-dir <dir>      Output directory (default: $WCAG_OUTPUT_DIR or ./audit)
  --rule-map <path>       Path to rule map JSONC file (default: <project-root>/wcag-rule-map.example.jsonc)
  --project-root <dir>    Project root (default: $WCAG_PROJECT_ROOT or cwd)
  --quiet                 Suppress progress output
  --verbose               Show debug-level output
```

### `wcag-audit extract-routes`

Extract routes from a project's file structure (utility, not part of main pipeline).

```
Options:
  --project-root <dir>    Project root to scan
  --output-dir <dir>      Where to write routes.json
  --routes-file <path>    Output filename (default: routes.json)
  --quiet                 Suppress progress output
  --verbose               Show debug-level output
```

---

## Phases

### Phase 0 — Scaffolding & Backup ✅

**Goal:** Set up the project skeleton. Nothing runs yet except `--help`.

**Steps:**

- [x] 0.1 — Create `rough_draft/` directory
- [x] 0.2 — Copy all source files into `rough_draft/`
- [x] 0.3 — Create the full directory tree under `src/`
- [x] 0.4 — Create `jsconfig.json` (later fixed: `module: "node16"` instead of `"es2022"`)
- [x] 0.5 — Update `package.json`
- [x] 0.6 — Create `bin/wcag-audit.js` with Commander skeleton
- [x] 0.7 — `yarn install`

**Commit:** `98d9751` — `"chore: scaffold CLI skeleton, back up rough draft"`

---

### Phase 1 — Shared Utilities & Constants ✅

**Goal:** Extract all shared code. No commands work yet, but all utility modules are importable with no side effects.

**Steps:**

- [x] 1.1–1.7 — All `src/utils/` modules created (logger, output, paths, slugs, text, browser, barrel)
- [x] 1.8–1.9 — `src/constants/wcag.js` + barrel
- [x] 1.10–1.13 — `src/config/` modules (audit-defaults, report-defaults, loader, barrel)

**Commit:** `8ae7e0d` — `"feat: extract shared utils, constants, and config modules"`

---

### Phase 2 — Engine Modules & False-Positive Filtering ✅

**Goal:** Each scan engine is independently importable. False-positive filtering is toggleable. All receive `ctx`.

**Steps:**

- [x] 2.1–2.2 — `src/filters/false-positives.js` + barrel
- [x] 2.3–2.6 — All `src/engines/` modules (ibm, axe, lighthouse, barrel)

**Commit:** `b984677` — `"feat: modular scan engines with toggleable false-positive filters"`

---

### Phase 3 — Server Lifecycle & Scanner ✅

**Goal:** The `audit` command is fully functional end-to-end.

**Steps:**

- [x] 3.1–3.2 — `src/server/lifecycle.js` + barrel
- [x] 3.3–3.6 — All `src/scanner/` modules (interactive-states, iframe-scanner, route-scanner, barrel)
- [x] 3.7–3.8 — `src/commands/audit.js` wired into `bin/wcag-audit.js`

**Commit:** `c459aeb` — `"feat: audit command fully functional"`

---

### Phase 4 — Report Generation ✅

**Goal:** The `report` command produces all 6 output files, including the new Third-Party Disclaimers section.

**Steps:**

- [x] 4.1–4.6 — All `src/report/` modules (builder, styles.css, html-generator, csv-generator, linear-export, barrel)
- [x] 4.7–4.8 — `src/commands/report.js` wired into `bin/wcag-audit.js`

**Commit:** `b5a1a35` — `"feat: report command with modular generators and third-party disclaimers"`

---

### Phase 5 — Route Extraction & Documentation ✅

**Goal:** The `extract-routes` subcommand works. Documentation is updated.

**Steps:**

- [x] 5.1 — All `src/extractors/` modules (detect, expo, vite, react-router, navigation, barrel)
- [x] 5.2–5.3 — `src/commands/extract-routes.js` wired into `bin/wcag-audit.js`
- [x] 5.4 — README.md updated

**Commit:** `a17da23` — `"feat: extract-routes command, updated docs"`

---

### Phase 6 — Cleanup & CI Readiness ✅

**Goal:** Remove old top-level scripts. Verify the full pipeline works end-to-end from a clean state.

**Steps:**

- [x] 6.1 — Old top-level scripts removed via `git rm`
- [x] 6.2 — `package.json` cleaned (deps moved from devDependencies to dependencies)
- [x] 6.3 — Full end-to-end pipeline tested (36 routes, 672 findings, all report outputs verified)

**Commit:** `836cb6b` — `"chore: remove old scripts, CI-ready"`

---

### Post-Phase Fix — Type Errors ✅

Resolved all 23 `checkJs` type errors across 4 files:
- `jsconfig.json`: Changed `module: "es2022"` → `module: "node16"` (was conflicting with `moduleResolution: "node16"`)
- `src/config/loader.js`: Cast validated config to `any` after guard check
- `src/engines/ibm.js`: Cast IBM checker config and report to `any` for strict enum types
- `src/server/lifecycle.js`: Cast `ChildProcess` for `.once()`, added `appReachable?` to timeouts typedef

**Commit:** `4ab0f6d` — `"fix: resolve all checkJs type errors"`

---

## Slug Redesign

### Before (ugly)

```
/                     → _home         (leading underscore)
/invoice/:id/payment  → invoice__dyn-id__payment  (dyn- prefix, double underscores)
/flow/accomodations-request/variant/main → flow__accomodations-request__variant__main
```

### After (clean dashes)

```
/                     → home
/invoice/:id/payment  → invoice-id-payment
/flow/accomodations-request/variant/main → flow-accomodations-request-variant-main
/billing              → billing
```

Algorithm:
1. Strip leading `/`
2. Remove `:` from dynamic segments (`:id` → `id`)
3. Replace `/` and any non-alphanumeric-or-hyphen characters with `-`
4. Collapse consecutive `-` to single `-`
5. Trim leading/trailing `-`
6. Fallback to `home` if empty

IBM report filenames follow the same convention:
- Base scan: `{slug}__base.html`
- Interactive state: `{slug}__{state}.html`
- iFrame: `{slug}__{state}__iframe-{index}.html`

Note: The `__` separator between slug/state/iframe parts is retained for parsability — only the route portion of the slug gets the clean-dash treatment.

---

## False-Positive Filter Design

```js
// src/filters/false-positives.js

export function createFilter(enabled = true) {
  if (!enabled) {
    return {
      filterIbm: (entries) => entries,
      filterAxe: (violations) => violations,
      patchIbmHtml: () => {},
    };
  }

  return {
    filterIbm: filterIbmFalsePositives,
    filterAxe: filterAxeFalsePositives,
    patchIbmHtml: patchIbmHtmlReport,
  };
}
```

Usage in engine modules:
```js
// src/engines/ibm.js
export async function runIbmScan(ctx, { page, url, label }) {
  const result = await ibmChecker.getCompliance(page, label);
  const violations = ctx.filter.filterIbm(result.report.results.filter(...));
  ctx.filter.patchIbmHtml(htmlPath);
  ctx.log.debug(`IBM: ${violations.length} violations after filtering`);
  // ...
}
```

Toggle from CLI: `--no-filter-fp` passes `createFilter(false)` → all filter functions become pass-through.

---

## Logger Design

```js
// src/utils/logger.js

export function createLogger({ quiet = false, verbose = false } = {}) {
  return {
    info:    quiet   ? () => {} : (...args) => console.log(...args),
    debug:   verbose ? (...args) => console.log('  ', ...args) : () => {},
    warn:    (...args) => console.warn('⚠', ...args),
    error:   (...args) => console.error('✖', ...args),
    success: quiet   ? () => {} : (msg) => console.log('✓', msg),
  };
}
```

Usage: `ctx.log.info(...)`, `ctx.log.debug(...)`, `ctx.log.success(path)`.

---

## Output Writer Design

```js
// src/utils/output.js
import fs from 'node:fs';
import path from 'node:path';

export function writeOutputs(outputDir, manifest, log) {
  fs.mkdirSync(outputDir, { recursive: true });
  for (const [name, content] of Object.entries(manifest)) {
    const filePath = path.join(outputDir, name);
    fs.writeFileSync(filePath, content);
    log.success(filePath);
  }
}
```

Usage in command handlers:
```js
writeOutputs(ctx.outputDir, {
  'wcag-conformance.json': JSON.stringify(report, null, 2),
  'wcag-conformance.csv': generateCsv(report),
  'wcag-conformance.html': generateHtml(report),
  'wcag-linear-issues.json': JSON.stringify(linearPayload, null, 2),
  'wcag-linear-issues.csv': linearPayload.csv,
  'wcag-jira-import.csv': linearPayload.jiraCsv,
}, ctx.log);
```

---

## Third-Party Disclaimers Section

Added to the HTML conformance report as a formal section after the per-criterion findings. Generated from `thirdPartyVendors` in the JSONC rule map config.

For each vendor, the disclaimer includes:

> **Third-Party Component Disclaimer: [Vendor Name]**
>
> This application integrates with [Vendor Name] via embedded iFrame(s) on the following domains: [domain patterns]. During accessibility testing, [N] findings were identified that originate from [Vendor Name]'s platform and cannot be remediated in this codebase.
>
> **Affected WCAG Criteria:** [list]
>
> **Issues:**
> - [Issue description, user impact, recommended fix]
>
> **Vendor Communication Status:** [status] (reported [date])
>
> These findings are documented for audit transparency but are excluded from this application's conformance scope. They are tracked separately for vendor communication and resolution.

---

## Future Considerations (Not in scope for this redesign)

These are documented for awareness but will be separate work:

1. **Playwright integration** — Extend the scanner to run checks on a pre-written Playwright test suite. The modular engine design and context pattern make this straightforward: create a `src/commands/playwright-audit.js` that receives page handles from Playwright instead of Puppeteer.

2. **Fast CI mode** — Engine selection (`--engines ibm,axe`) and changed-routes-only scanning for pull request CI. The `scanRoutes(ctx, routes)` function already receives its engine list on the context, so this is additive.

3. **`npx` support** — Once stable, publish to npm for `npx wcag-audit audit --route /welcome` usage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JasonMaggard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
