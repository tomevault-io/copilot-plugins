## webqualityanalyzer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT

Always ignore files in the .gitignore file unless specifically asked to read them.

## Commands

```bash
npm run build          # Compile TypeScript and bundle for both browsers → dist/chrome/ and dist/firefox/
npm run build:chrome   # Chrome only
npm run build:firefox  # Firefox only
npm run dev            # Watch mode (Chrome)
npm run dev:firefox    # Watch mode (Firefox)
npm test               # Run Jest test suite
npm run test:watch     # Jest in watch mode
npm run lint           # ESLint on src/ and tests/
```

To run a single test file:

```bash
npx jest tests/unit/content.core.test.ts
```

## Architecture

WebQualityAnalyzer is a **Chrome and Firefox Manifest V3 browser extension** with three isolated execution contexts, each compiled to a separate Webpack bundle:

| Script                         | Bundle                 | Role                                                         |
| ------------------------------ | ---------------------- | ------------------------------------------------------------ |
| `src/background/background.ts` | `background.bundle.js` | Extension lifecycle events                                   |
| `src/content/content.ts`       | `content.bundle.js`    | Content script — orchestrates analysis and registers message listener |
| `src/popup/popup.ts`           | `popup.bundle.js`      | Popup UI — sends messages to content script, renders results |
| `src/shared/browser.ts`        | (imported by all)      | Re-exports `webextension-polyfill` as `browser.*`            |
| `src/shared/settings.ts`       | (imported by all)      | `AnalyzerSettings` interfaces, `DEFAULT_SETTINGS`, `STORAGE_KEY` |

### Content module layout

`src/content/` is split into focused modules:

- `content.ts` — thin orchestrator: registers `onMessage` listener, calls analyzers, re-exports types
- `types.ts` — `AnalysisResult`, `CategoryResult`, `Issue` interface definitions
- `utils.ts` — `getCssSelector` and `getHtmlSnippet` DOM helpers used by analyzers
- `analyzers/accessibility.ts` — missing alt text, unlabelled inputs, heading hierarchy
- `analyzers/seo.ts` — title length, meta description, H1, canonical, Open Graph
- `analyzers/performance.ts` — oversized images, lazy-loading, external resources, inline styles

### Popup module layout

`src/popup/` is split into:

- `popup.ts` — UI logic: tab switching, result rendering, expandable issue panels, settings UI wiring (`initSettings`, `populateSettingsUI`, `collectSettings`, `attachSettingsListeners`)
- `popup.css` — all popup styles (no inline CSS)
- `utils.ts` — `escapeHtml`, `getScoreColor`, `exportResults`
- `settings.ts` — `loadSettings`, `saveSettings`, `resetSettings` using `browser.storage.local`; deep-merges stored values with `DEFAULT_SETTINGS` on load to handle schema evolution

### Shared module layout

`src/shared/` is imported by both the popup and content bundles:

- `browser.ts` — re-exports `webextension-polyfill` as `browser.*`
- `settings.ts` — `AnalyzerSettings`, `AccessibilitySettings`, `SeoSettings`, `PerformanceSettings` interfaces; `DEFAULT_SETTINGS` constant (mirrors all hardcoded analyzer thresholds); `STORAGE_KEY` constant

### Communication Flow

Popup → (`browser.tabs.sendMessage` with `{ action: 'analyze', settings: AnalyzerSettings }`) → Content Script → returns `AnalysisResult` → Popup renders

The popup reads settings from `browser.storage.local` before each analysis and sends them in the message payload. The content script is stateless — it uses the settings for that one analysis and discards them. If `settings` is absent (older popup), the content script falls back to `DEFAULT_SETTINGS`.

### Key Interfaces (defined in `src/content/types.ts`, re-exported from `src/content/content.ts`)

```typescript
export interface AnalysisResult {
  score: number; // 0–100 overall (mean of three categories)
  pageInfo: {
    url: string;
    title: string;
    timestamp: string; // ISO 8601
  };
  categories: {
    accessibility: CategoryResult;
    seo: CategoryResult;
    performance: CategoryResult;
  };
}

export interface CategoryResult {
  score: number; // 0–100
  issues: Issue[];
  suggestions: string[];
}

export interface Issue {
  type: string; // e.g. 'Missing Alt Text', 'Page Title'
  message: string;
  severity: 'high' | 'medium' | 'low';
  selector?: string;     // CSS path built by getCssSelector()
  htmlSnippet?: string;  // outer HTML snippet built by getHtmlSnippet()
}
```

Other bundles consume them with zero runtime cost via `import type { AnalysisResult, CategoryResult } from '../content/content'` — types are erased before bundling.

### Build Output

Webpack accepts `--env browser=chrome|firefox` and outputs to `dist/<browser>/`. The correct manifest (`src/manifest.chrome.json` or `src/manifest.firefox.json`) and `src/popup.html` are copied alongside the bundles.

- `dist/chrome/` — load via `chrome://extensions` → Load unpacked
- `dist/firefox/` — load via `about:debugging` → Load Temporary Add-on → select `manifest.json`

### Browser API

All extension API calls use `browser.*` imported from `src/shared/browser.ts`, which re-exports `webextension-polyfill`. This provides a unified Promise-based API across Chrome and Firefox — no `chrome.*` calls appear in source files directly.

The `onMessage` listener in `content.ts` returns a `Promise` (polyfill pattern) rather than using the `sendResponse` callback.

## Code Quality

- **TypeScript strict mode** is on — `noImplicitReturns`, `noUnusedLocals`, explicit return types preferred
- **ESLint rules**: `prefer-const`, `no-var`, `@typescript-eslint/no-unused-vars` all set to `error`
- **Prettier**: single quotes, 2-space indent, trailing commas (ES5), 80-char print width
- **Jest coverage threshold**: 80% across branches, functions, lines, and statements

## CI

| Workflow | File                          | Triggers                       | Steps              |
| -------- | ----------------------------- | ------------------------------ | ------------------ |
| CI       | `.github/workflows/ci.yml`    | push (all branches), PR → main | lint, build        |
| Tests    | `.github/workflows/tests.yml` | push (all branches), PR → main | test with coverage |

Actions are pinned to commit SHAs for supply-chain security. Dependabot is configured to keep them up to date weekly.

## Testing

### Test files

| File                                       | What it covers                                                                                                      |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| `tests/unit/content.core.test.ts`          | `performQualityAnalysis`, `browser.runtime.onMessage` listener, `getCssSelector`, `getHtmlSnippet`                 |
| `tests/unit/content.accessibility.test.ts` | `analyzeAccessibility` — all accessibility checks                                                                   |
| `tests/unit/content.seo.test.ts`           | `analyzeSEO` — all SEO checks                                                                                       |
| `tests/unit/content.performance.test.ts`   | `analyzePerformance` — all performance checks                                                                       |
| `tests/unit/popup.actions.test.ts`         | `runAnalysis`, `exportResults`, `switchTab`                                                                         |
| `tests/unit/popup.display.test.ts`         | `displayResults`, `displayCategoryContent`, `showLoadingState`, `showError`, expandable issue panels                |
| `tests/unit/popup.ui.test.ts`              | `updatePageInfo`, `getScoreColor`, score color thresholds                                                           |
| `tests/unit/background.test.ts`            | `browser.runtime.onInstalled` registration and callback                                                             |
| `tests/unit/settings.storage.test.ts`      | `loadSettings`, `saveSettings`, `resetSettings` — empty storage → defaults, partial merge, save, reset             |
| `tests/unit/settings.ui.test.ts`           | `populateSettingsUI`, `collectSettings`, `attachSettingsListeners` — populate, collect, auto-save on change, toggle disabled state, stopPropagation on toggle click, reset button |
| `tests/helpers/helpers.ts`                 | Shared DOM helpers for content tests: `appendImg`, `appendInput`, `appendHeading`, `appendMeta`, `appendLink`, `appendH1`, `addImageWithDimensions`, `appendImgs`, `appendSpansWithStyle`, `appendExternalScripts`, `appendExternalLinks`, `appendExternalAnchors` |
| `tests/helpers/popup.ts`                   | Shared popup fixtures: `buildCategory`, `buildResult`, `setupPopupDOM` (includes all settings DOM elements)         |

Coverage baseline (2026-03): **95.14% stmts / 88.23% branches / 88.33% funcs / 95.56% lines** — all above the 80% global threshold.

### Setup

`webextension-polyfill` is intercepted in tests via `moduleNameMapper` in `jest.config.ts`, which redirects it to `tests/__mocks__/webextension-polyfill.ts`. That stub re-exports the global `chrome` mock, so `browser.*` calls in source files resolve to the same `jest.fn()` stubs.

Chrome extension APIs (`chrome.*`) are mocked globally in `tests/setup.ts` (loaded via `setupFilesAfterEnv`). All `chrome.runtime`, `chrome.tabs`, and `chrome.storage.local` methods are `jest.fn()` so any module that calls them at import time won't throw in jsdom.

### Key patterns

- **Module load-time side effects** (background.ts, content.ts): capture the registered listener in `beforeAll` before any `jest.clearAllMocks()` in `beforeEach` wipes it. For background.ts use `jest.isolateModules` + `require()` to get a fresh execution per test — suppress the lint rule with `// eslint-disable-next-line @typescript-eslint/no-require-imports` on that line.
- **onMessage listener signature**: the listener extracts `settings` from `{ action: 'analyze', settings? }`, falls back to `DEFAULT_SETTINGS`, and returns `Promise.resolve(performQualityAnalysis(settings))` — test it by awaiting the return value, not by checking a `sendResponse` callback.
- **Settings storage in tests**: `chrome.storage.local.get` must be mocked with `.mockResolvedValue({})` in `beforeEach` for any test that triggers `loadSettings()` (including `runAnalysis` tests). `jest.clearAllMocks()` wipes return values, so set the mock after clearing.
- **Settings UI auto-save**: number inputs fire `saveSettings` on the `change` event; toggle checkboxes fire it on `change` too (plus update the `.disabled` class on the section body). The reset button is async — flush with `await new Promise(resolve => setTimeout(resolve, 0))` to wait for the full chain before asserting.
- **"Perfect Score!" condition**: `displayCategoryContent` shows it only when `issues.length === 0 && suggestions.length === 0`. Performance generic suggestions are only emitted when `score < 100`, so all three category tabs can reach this state.
- **popup.css import**: `popup.ts` imports `./popup.css`; tests mock CSS modules via `moduleNameMapper` in `jest.config.ts` pointing to `tests/__mocks__/style.mock.ts`.
- **DOM helpers**: all reusable DOM-construction functions live in `tests/helpers/helpers.ts`; popup fixtures (`buildCategory`, `buildResult`, `setupPopupDOM`) live in `tests/helpers/popup.ts`. Import from there rather than defining locally.
- **Security hook in tests**: a `PreToolUse:Edit` hook rejects edits that set DOM content via direct HTML string assignment. Use `document.createElement` + `appendChild` when adding test DOM nodes; use `element.replaceChildren()` to clear rather than `innerHTML = ''`.
- **jsdom quirks**:
  - Any head manipulation (`appendChild`, `replaceChildren`) resets `document.title` — always set `document.title` _after_ modifying head children.
  - `img.naturalWidth` / `img.naturalHeight` are always `0` — use `Object.defineProperty` with a getter to simulate large images.
  - `Blob.prototype.text()` is not implemented — spy on `JSON.stringify` to inspect data passed to the Blob constructor.
  - `URL.createObjectURL` / `URL.revokeObjectURL` are not implemented — mock both on `global.URL` before testing `exportResults`.
  - `element.style.background` is normalized to `rgb()` — use `element.getAttribute('style')` to assert raw hex values set via inline styles.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adrianjiga)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adrianjiga)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
