## frontend

> This document defines coding conventions for AI agents working in this repository.

# Agent Instructions

This document defines coding conventions for AI agents working in this repository.

## Core Principles

- **Vanilla only** — use plain HTML, CSS, and JavaScript. Do NOT introduce any frontend frameworks (React, Vue, Angular, Svelte, etc.) or TypeScript.
- **Bootstrap theme** — all UI must use Bootstrap 5 classes and the project's custom SCSS theme. Do not add Tailwind, Material UI, or any other CSS framework.
- **No additional JS libraries** — avoid adding new npm dependencies unless explicitly requested. Use native browser APIs (`fetch`, `document.querySelector`, `addEventListener`, etc.) instead of jQuery or utility libraries.

## Technology Stack

- **Build tool**: Vite
- **CSS**: Bootstrap 5.3 with custom SCSS variables (`src/scss/_variables.scss`)
- **Icons**: Font Awesome 6 (free) — use `<i class="fa-solid fa-*">`, `<i class="fa-regular fa-*">`, or `<i class="fa-brands fa-*">` classes. Do NOT use Bootstrap Icons or inline SVGs.
- **JS**: Vanilla ES modules (`type: "module"`)
- **Alerts/Toasts**: Bootstrap native toasts
- **Package manager**: npm

## File Structure Conventions

- HTML pages go in `src/` (e.g. `src/my-page.html`). Vite auto-discovers them.
- Each page has a dedicated folder: `src/page_assets/<page-name>/`.
  - Entry JS: `src/page_assets/<page-name>/js/main.js` (referenced by the HTML `<script>` tag).
  - Additional JS modules: `src/page_assets/<page-name>/js/<script-name>.js`.
  - Page CSS: `src/page_assets/<page-name>/css/main.css`.
- `src/common/` — **READ-ONLY**. Contains shared utilities (`api.js`, `bsToast.js`, `dom.js`) and shared CSS (`custom.css`). Do NOT add or modify files here.
- `src/scss/` — **READ-ONLY**. Do not add or modify SCSS files.
- Static assets (images, fonts) go in `src/public/`.
- Icons: use Font Awesome classes (`fa-solid`, `fa-regular`, `fa-brands`). Do NOT add icon image files or SVG sprite sheets.

## Folder Isolation Rules

- **`src/common/` and `src/scss/` are READ-ONLY.** Never create, edit, or delete files in these folders.
- **All new JS and CSS must be page-specific.** Place it under `src/page_assets/<page-name>/js/` or `src/page_assets/<page-name>/css/` — never in `src/common/`.
- Do not create catch-all or "shared" files (e.g. `utils.js`, `helpers.css`) inside a new page folder either — split logic into focused, well-named modules within that page's `js/` subfolder.
- `src/common/` utilities may be *imported* by page scripts but must not be modified.

## Styling Rules

- Always use Bootstrap utility classes first (e.g. `mt-3`, `d-flex`, `text-muted`).
- **Write custom styles as plain CSS only.** `src/scss/` is READ-ONLY in this repository, so do not add new SCSS files.
- All custom CSS must be page-specific: `src/page_assets/<page-name>/css/main.css`. Do NOT add styles to `src/common/css/`.
- Use Bootstrap's CSS custom properties (`var(--bs-primary)`, `var(--bs-body-bg)`, etc.) instead of hardcoding colors or spacing values.
- Do NOT write new SCSS component files under `src/scss/components/`.
- Keep custom CSS minimal; prefer Bootstrap's built-in classes.
- All buttons should use Bootstrap button classes (`btn btn-primary`, `btn-outline-danger`, etc.).

### Linking CSS in the entry JS file

```js
import * as bootstrap from 'bootstrap/dist/js/bootstrap.bundle.min.js';
window.bootstrap = bootstrap;
import '../../../scss/styles.scss';        // Bootstrap + SCSS theme
import '../../../common/css/custom.css';   // existing shared overrides (READ-ONLY — do not edit)
import '../css/main.css';           // page-specific styles
```

## JavaScript Rules

- Use ES module `import`/`export` syntax.
- **`main.js` is the sole entry point** for each page. It is the only file referenced by the HTML `<script>` tag.
- **Additional JS for a page** must be split into focused modules at `src/page_assets/<page-name>/js/<script-name>.js` and imported by `main.js`. Examples: `table.js`, `form.js`, `chart.js`.
- **Do NOT add new files to `src/common/js/`.** It is READ-ONLY. Only import from it.
- Use the existing common utilities where applicable (import-only — do not modify):
  - `src/common/js/api.js` — for all HTTP requests (uses `VITE_API_BASE_URL`).
  - `src/common/js/bsToast.js` — for Bootstrap native toast notifications.
  - `src/common/js/dom.js` — for DOM helpers (`$`, `$$`, `on`, `off`, `ready`).
- Do not use `var`. Use `const` by default, `let` only when reassignment is needed.
- Use `async`/`await` over `.then()` chains.
- `main.js` must always start with the Bootstrap + SCSS imports:
  ```js
  import * as bootstrap from 'bootstrap/dist/js/bootstrap.bundle.min.js';
  window.bootstrap = bootstrap;
  import '../../../scss/styles.scss';
  import '../../../common/css/custom.css'; // READ-ONLY
  import '../css/main.css';
  ```
  Then import your page modules:
  ```js
  import './table.js';
  import './form.js';
  ```

## Adding a New Page

1. Create `src/<page-name>.html` with the standard `<head>` (charset, viewport, title, favicon, meta tags).
2. Create `src/page_assets/<page-name>/js/main.js` — Bootstrap + SCSS + CSS imports, then import page modules.
3. Create `src/page_assets/<page-name>/css/main.css` — page-specific styles.
4. For each logical concern (table, form, chart, etc.) create `src/page_assets/<page-name>/js/<script-name>.js` and import it from `main.js`.
5. Reference only the entry script in HTML: `<script type="module" src="/page_assets/<page-name>/js/main.js"></script>`.
6. Reuse the navbar and footer markup from `index.html`.

Expected folder layout for a page named `users`:
```text
src/
  users.html
  page_assets/
    users/
      js/
        main.js       ← entry point (Bootstrap imports + module imports)
        table.js      ← table logic
        form.js       ← form/validation logic
      css/
        main.css     ← page-specific styles
```

## Environment Variables

- All environment variables must be prefixed with `VITE_`.
- Access them via `import.meta.env.VITE_*`.
- Never hardcode API URLs — always use `VITE_API_BASE_URL` through the `api.js` client.

## Linting & Formatting

- Run `npm run lint` and `npm run lint:css` before considering work complete.
- Follow the ESLint, Prettier, and Stylelint configs in the project root.

---
> Source: [supply-chain-lite/frontend](https://github.com/supply-chain-lite/frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
