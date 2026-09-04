## bootstrap-admin-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Modern Bootstrap 5 Admin Dashboard Template (v3.4.0) using Vite, Alpine.js, and SCSS.

| Directory | Purpose |
|-----------|---------|
| `src-modern/` | Source files (Bootstrap 5.3.8, ES6+ modules) |
| `dist-modern/` | Production build output |
| `src/`, `dist/` | Legacy Bootstrap 3 code - **do not use** |

## Commands

```bash
npm run dev        # Dev server at http://localhost:3000
npm run build      # Production build
npm run lint       # ESLint check
npm run format     # Prettier format
npm run test:build # Build, then smoke-test all 21 pages in a headless browser
npm run verify     # lint + audit + test:build — run before shipping
```

The smoke test needs Playwright, which is deliberately not a dependency:
`npx playwright install chromium && PLAYWRIGHT_PATH=playwright npm run test:build`.

## Key Files

| File | Purpose |
|------|---------|
| `src-modern/scripts/main.js` | App entry point, initializes AdminApp class |
| `src-modern/styles/scss/main.scss` | SCSS entry point |
| `vite.config.js` | Build config with multi-page entries |
| `eslint.config.js` | ESLint v10 flat config |
| `src-modern/scripts/utils/search-component.js` | Factory for the navbar search Alpine component |
| `src-modern/scripts/utils/constants.js` | Shared timing/breakpoint constants |
| `src-modern/scripts/utils/apex.js` | Modular ApexCharts entry — core + only the chart types used |
| `src-modern/scripts/utils/prism.js` | Bundled syntax highlighting for the element showcase pages |
| `scripts/smoke-test.mjs` | Loads all 21 built pages in a browser; fails on errors or external requests |
| `src-modern/styles/scss/components/_a11y.scss` | Skip-link, reduced-motion, sr-only helpers |
| `src-modern/styles/scss/components/_bootstrap-icons-subset.scss` | Generated icon CSS (only the ~158 used) |

## Architecture Quick Reference

**Page Detection**: Each HTML page needs `data-page="pagename"` on `<body>` to load correct component.

**Component Loading**: Dynamic imports in `main.js` based on page:

```javascript
case 'users':
  await import('./components/users.js');
  break;
```

**Alpine.js Pattern**: Components register via `Alpine.data()`:

```javascript
Alpine.data('componentName', () => ({
  init() { /* setup */ },
  // methods and state
}));
```

## Adding a New Page

1. Create `src-modern/newpage.html` with `data-page="newpage"` on body
2. Add entry to `vite.config.js` → `rollupOptions.input`
3. Create `src-modern/scripts/components/newpage.js`
4. Add case to `initPageComponents()` in `main.js`
5. Optional: Create `src-modern/styles/scss/pages/_newpage.scss` and import in `main.scss`

## Dependencies

- **UI**: Bootstrap 5.3.8, Bootstrap Icons 1.13.1 (subset, see below)
- **Reactive**: Alpine.js 3.15.12
- **Charts**: ApexCharts 6.7 (single chart library — Chart.js was removed in v3.4.0)
- **Notifications**: SweetAlert2 11.26.25
- **Highlighting**: Prism 1.30 (bundled, showcase pages only)
- **Fonts**: Inter via `@fontsource-variable/inter` (self-hosted)
- **Build**: Vite 8.2 (rolldown), Sass 1.102

## Conventions

- **One charting library.** Use ApexCharts. ApexCharts requires a `<div>` container — not `<canvas>`.
- **Import ApexCharts from `utils/apex.js`, never from `'apexcharts'`.** That module registers only the chart types this template renders. Using a new chart type means adding its module there, or the chart silently fails to render.
- **No third-party runtime requests.** No CDN `<script>`/`<link>`, no remote fonts or images. `npm run test:build` fails the build if a page requests anything off-origin.
- **No inline `<script>` or `onclick`.** Use a delegated listener in the page's component keyed off a `data-*` attribute. This keeps the template CSP-compatible and the code lintable — six pages once shipped a silent SyntaxError precisely because the logic sat in inline scripts nobody linted.
- **Nothing is a global.** Alpine, ApexCharts, Swal and Bootstrap are all ES module imports. ESLint declares no library globals, so a missing import fails `npm run lint`.
- **Register Alpine components under the name the markup calls.** `x-data="contactForm()"` needs `Alpine.data('contactForm', …)`. A mismatch throws on every binding in that scope and is invisible until the page is opened.
- **Guard `DOMContentLoaded` in page components.** They are dynamically imported and usually resolve after the event has fired — check `document.readyState` first.
- **No `innerHTML` with interpolation.** Use `createElement` + `textContent`. Toast `action` callbacks must be functions, not strings.
- **Search inputs** use the shared factory: `Alpine.data('searchComponent', createSearchComponent({ getResults }))`. Don't paste fresh definitions per page.
- **Component cleanup.** Anything that calls `setInterval` / `setTimeout` / `addEventListener` should track the IDs and clean up in `destroy()` (called via `pagehide`).
- **Bootstrap Icons subset.** When adding an icon used in markup that wasn't in the subset, regenerate `_bootstrap-icons-subset.scss` (script in CHANGELOG 3.4.0 entry / see node script below) so the icon's `content: "\..."` rule exists.

## Documentation

See [DEVELOPMENT.md](DEVELOPMENT.md) for detailed architecture, patterns, and styling guide.

---
> Source: [puikinsh/Bootstrap-Admin-Template](https://github.com/puikinsh/Bootstrap-Admin-Template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
