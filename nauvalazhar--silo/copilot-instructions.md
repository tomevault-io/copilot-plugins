## silo

> This file is the single source of truth for Codex when working in this repository. Every code generation, refactor, or review **must** follow these instructions exactly.

# AGENTS.md — Master Instruction

This file is the single source of truth for Codex when working in this repository. Every code generation, refactor, or review **must** follow these instructions exactly.

## Project Identity

**Silo** — a Chrome extension that audits the SEO health of any web page during local development. Built for developers who want Lighthouse-grade SEO feedback in a persistent side panel while they build.

**Tech stack**: React 19, TypeScript, Vite 7, Tailwind CSS 4, CRXJS Vite Plugin, Manifest V3.

## Baseline Standard

All audit logic **must** follow the standards defined by Google Lighthouse SEO Audits and the referenced accessibility rules. The Master Audit Registry below is the **absolute reference** — every validator's `sourceName` and `sourceUrl` must match it exactly. **Any deviation from these sources is not allowed unless explicitly requested.**

**Every `ValidationItem` must include source attribution:**

```ts
interface ValidationItem {
  severity: Severity          // 'ok' | 'warning' | 'error' | 'critical'
  message: string             // What is wrong (or right)
  advice: string              // Why it matters
  fix: string                 // How to fix it
  sourceName: string          // Must match the registry below
  sourceUrl: string           // Must match the registry below
}
```

## Master Audit Registry

This is the official mapping. When building or updating any validator, use the `sourceName` and `sourceUrl` from this table verbatim.

- [x] **1. `crawlability`** — Page isn't blocked from indexing · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/is-crawlable`
- [x] **2. `title`** — Document has a `<title>` element · Deque University · `https://dequeuniversity.com/rules/axe/4.11/document-title`
- [x] **3. `description`** — Document has a meta description · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/meta-description`
- [ ] **4. `http-status`** — Page has successful HTTP status code (200 OK) · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/http-status-code/` *(skipped — not actionable in extension context; page is always loaded)*
- [x] **5. `link-text`** — Links have descriptive text (avoid "click here") · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/link-text/`
- [x] **6. `crawlable-links`** — Links must use proper `href` attributes · Google Search · `https://developers.google.com/search/docs/crawling-indexing/links-crawlable`
- [x] **7. `robots-txt`** — `robots.txt` is valid and well-formed · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/invalid-robots-txt/`
- [x] **8. `alt-text`** — Image elements have `[alt]` attributes · Deque University · `https://dequeuniversity.com/rules/axe/4.11/image-alt`
- [x] **9. `hreflang`** — Document has a valid `hreflang` for multi-language · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/hreflang/`
- [x] **10. `canonical`** — Document has a valid `rel=canonical` · Lighthouse · `https://developer.chrome.com/docs/lighthouse/seo/canonical/`
- [x] **11. `viewport`** — Page has a `<meta name="viewport">` tag with `width=device-width` · Lighthouse · `https://developer.chrome.com/docs/lighthouse/pwa/viewport/`
- [x] **12. `structured-data`** — JSON-LD structured data is valid and well-formed · Google Search Central · `https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data`
- [x] **13. `sitemap`** — `sitemap.xml` is valid and well-formed · Google Search Central · `https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview`

## Audit Roadmap

### Phase 1 — Lighthouse Compliance (The 10 Core Audits)

Implement all 10 audits from the Master Audit Registry above. Each must return `ValidationItem` objects with the exact `sourceName` and `sourceUrl` from the registry.

- [x] `title` — Done
- [x] `description` — Done
- [x] `canonical` — Done
- [x] `alt-text` — Done (retrofitted with source attribution)
- [x] `crawlable-links` — Done (retrofitted with source attribution)
- [x] `link-text` — Done
- [x] `crawlability` — Done (meta robots + X-Robots-Tag header via background service worker)
- [ ] `http-status` — Skipped (not actionable in extension context; page is already loaded)
- [x] `robots-txt` — Done (fetched from side panel)
- [x] `hreflang` — Done

### Phase 2 — Developer Utilities ✅

- [x] Real-time Broken Link Checker
- [x] Localhost Absolute Link Detector
- [x] Heading Tree Visualizer

### Phase 3 — Content Intelligence ✅

- [x] Flesch Reading Ease scoring
- [x] One-click copy fix in AuditAdvice
- [x] Scan diff / regression tracking
- [x] Editable social preview + OG code generator
- [x] Keyword analysis

### Phase 4 — AI-Powered Features ✅

- [x] API key management + settings UI
- [x] AI meta title/description generator
- [x] AI alt text generator (Codex vision)
- [x] AI structured data (JSON-LD) generator
- [x] Content ↔ meta relevance score
- [x] Tone analysis

### Phase 5 — Power User Features ✅

- [x] SEO history timeline (SVG sparkline)
- [x] HTML export report

## Architecture

**Extension entry points** (defined in `manifest.config.ts`):

- **Popup** (`src/popup/`) — single button that opens the side panel via `chrome.sidePanel.open()`, then closes itself.
- **Side Panel** (`src/sidepanel/`) — the main UI. Sends a `SCAN_SEO` message to the content script on the active tab, validates the returned `SeoData`, and renders audit results.
- **Content Script** (`src/content/index.ts`) — injected into all pages. Handles `SCAN_SEO` messages and live DOM observation (`WATCH_CHANGES`/`STOP_WATCHING`) via `MutationObserver`.
- **Background Service Worker** (`src/background/index.ts`) — captures HTTP status codes and `X-Robots-Tag` response headers via `chrome.webRequest.onCompleted`. Stores per-tab and responds to `GET_PAGE_RESPONSE` messages.

**Data flow**:

```
Side Panel triggers a scan:
  1. { type: 'SCAN_SEO' } → Content Script → returns SeoData (DOM data incl. viewport, jsonLd, bodyText, wordCount)
  2. { type: 'GET_PAGE_RESPONSE' } → Background SW → returns { statusCode, xRobotsTag }
  3. fetch('/robots.txt') → Side Panel fetches directly from page origin
  4. fetch('/sitemap.xml') → Side Panel fetches directly from page origin
  → All results validated by validator/
  → Score computed by scoring.ts
  → rendered by section components (ScoreDashboard, StructuredDataAudit, etc.)
```

**Key detail**: `ensureContentScript()` in `App.tsx` PINGs the content script before every scan and programmatically injects it if missing, so the scan works even if the manifest auto-injection hasn't fired yet.

**Validation system** (`src/sidepanel/validator.ts`): Pure functions that take `SeoData` fields and return `ValidationItem` objects. `worstSeverity()` aggregates across items.

**Types**: SEO data interfaces in `src/types/seo.ts`. Validation types co-located in `validator.ts`.

**Side panel components** (`src/sidepanel/components/`): `ScoreDashboard`, `ScoreHistory`, `ScanDiff`, `MetaOverview`, `ContentQuality`, `KeywordAnalysis`, `SocialPreview`, `StructuredDataAudit`, `HeadingHierarchy`, `ImageAudit`, `LinksHealth`, `SettingsPanel`, `AiMetaGenerator`. Shared primitives: `StatusIcon`, `AuditAdvice`.

**AI system** (`src/sidepanel/ai/`): `config.ts` manages OpenRouter API key via `chrome.storage.sync`. `client.ts` wraps the OpenRouter Chat Completions API (`https://openrouter.ai/api/v1/chat/completions`). Feature modules: `metaGenerator.ts`, `altTextGenerator.ts`, `structuredDataGenerator.ts`, `relevanceScore.ts`, `toneAnalysis.ts`. All AI features are opt-in — gated by `hasApiKey` prop. Default model: `anthropic/Codex-haiku-4.5` (via OpenRouter).

**Content analysis** (`src/sidepanel/validator/readability.ts`, `keyword.ts`): Pure heuristic-based validators. Readability uses Flesch Reading Ease formula. Keyword detection finds common phrases between title and H1. Neither is a Tier 1 scoring audit.

**Scan diff** (`src/sidepanel/diff.ts`): Compares current vs. previous `AuditSnapshot` by `message` string identity. Shows score delta, new issues, and fixed issues.

**History** (`src/storage.ts`): `appendHistory` / `loadHistory` track score trends per page (max 30 entries/page, 20 pages) in separate `snapshotHistory` storage key.

**Export** (`src/sidepanel/exportReport.ts`, `exportHtmlReport.ts`): JSON report and self-contained HTML report with inline CSS, SVG score circle, and printable layout.

**Scoring** (`src/sidepanel/scoring.ts`): Calculates a 0–100 score from 9 Tier 1 audits. Each audit contributes equal weight; `ok` = full, `warning` = half, `error`/`critical` = 0.

**Export** (`src/sidepanel/exportReport.ts`): Builds a JSON report with score, validation results, and raw data summary. Downloads as `seo-report-{domain}-{date}.json`.

## Validator Contract

Every validation function **must** follow this contract:

1. **Return `ValidationItem`** with all six fields: `severity`, `message`, `advice`, `fix`, `sourceName`, `sourceUrl`.
2. **Source attribution is mandatory** — `sourceName` and `sourceUrl` must match the Master Audit Registry exactly. No validator ships without them.
3. **Match source criteria** — use the same pass/fail thresholds documented at the audit's `sourceUrl`.
4. **Pure functions only** — validators take data in, return results out. No side effects, no DOM access.
5. **Severity levels**:
   - `ok` — audit passes.
   - `warning` — non-critical issue, may affect ranking or UX.
   - `error` — significant issue, likely affects ranking.
   - `critical` — fundamental problem, page will fail the audit.

## Commands

- `bun install` — install dependencies
- `bun run dev` — start Vite dev server (load `dist/` as unpacked extension in Chrome)
- `bun run build` — type-check then build for production (outputs to `dist/`, zips to `release/`)
- No test runner is configured

## Conventions

- **Path alias**: `@/` maps to `src/` (configured in `vite.config.ts` and `tsconfig.app.json`)
- **Styling**: Tailwind CSS 4 utility classes only (`@import "tailwindcss"`)
- **Package manager**: bun (`bun.lock`)
- **Strict TypeScript**: `noUnusedLocals`, `noUnusedParameters`, `noEmit` enabled
- **No over-engineering**: validators are simple pure functions, not class hierarchies or plugin systems

---
> Source: [nauvalazhar/silo](https://github.com/nauvalazhar/silo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
