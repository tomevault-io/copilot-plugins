## batterup

> - Use `./deploy.sh` instead of `firebase deploy` — it auto-bumps the cache version (`?v=` timestamp on CSS/JS imports) so PWA users get fresh files immediately.

# BatterUp

## Deployment
- Use `./deploy.sh` instead of `firebase deploy` — it auto-bumps the cache version (`?v=` timestamp on CSS/JS imports) so PWA users get fresh files immediately.

## Project Structure
- `web/` — static PWA served by Firebase Hosting
- `web/js/` — vanilla ES modules (no build step)
- `web/js/constants.js` — shared constants with no Firebase dependency (keeps helpers testable in Node), including most `ICON_*` SVG strings used in JS-rendered markup — EXCEPT `ICON_EDIT`, `ICON_DELETE`, `ICON_WA`, which live in `state.js`
- `web/fonts/` — self-hosted Fraunces variable font (CSP is `font-src 'self'`, so no font CDNs); used via `var(--font-display)` for headings and large money figures
- `web/js/dispatch.js` — centralized event delegation (`act()`/`actOn()` + `data-action`)
- `web/js/tools/` — one module per Tools-tab feature (route, delivery-run, payments, materials, home-batter, price-summary, ingredients, notifications) plus the shared `message-overlay`; `tools.js` is just the tab shell (tile nav, summaries, past orders) and imports them. Pure logic lives in `helpers.js` (unit-tested); modules expose `window.*` handlers for `data-action` and `export` their render entry points. Add new module files to the `sw.js` SHELL precache list.
- Catalog is a Tools tile (`data-action="openTool"`, `data-args='["catalog"]'`, markup at `#tool-detail-catalog` in `index.html`) merging Customers + Products behind a segmented toggle; `switchCatalog`/`catalogAdd` live in `app.js`, list rendering in `customers.js`/`products.js`. Design rationale: `docs/superpowers/specs/2026-06-13-catalog-tab-design.md`.
- `web/js/labels.js` — batch thermal-label printing (40×30mm), allergen detection, multi-image share; imported by `orders.js` and `tools.js`
- `web/sw.js` — service worker; caches the app shell for offline (cache name bumped by `deploy.sh`)
- `web/css/styles.css` — all styles, with CSS variables for light/dark mode
- `firestore.rules` — Firestore security rules (whitelist-based)
- `firebase.json` — hosting config with security headers
- `tests/` — unit tests (`npm run test:unit`, `node --test` for pure helpers) and jsdom DOM smoke tests (`npm run test:dom`, harness in `tests/dom/`); both run in `npm run check` and `deploy.sh`

## Conventions
- All colors must use CSS variables (defined in `:root` and overridden in `@media (prefers-color-scheme: dark)`) — no hardcoded hex values in JS or HTML
- No emoji in UI chrome — use the stroke SVG icons (`ICON_*` in `constants.js`, or inline `<svg>` in `index.html`) so icons render consistently; emoji inside WhatsApp message text is fine
- Use `esc()` from `helpers.js` for all user-generated content rendered as HTML
- Use `shortName()` from `helpers.js` for displaying customer names (drops last name)
- Home orders (`is_home: true`) are excluded from Orders tab and Reports but included in Materials Needed
- Firestore writes for multiple orders should use `Promise.all`, not sequential awaits
- Use `localDateStr()` from `helpers.js` for YYYY-MM-DD date strings — never `toISOString().split("T")[0]`, which gives the UTC date and is wrong around midnight for non-UTC users
- No inline event handlers (`onclick=`, `oninput=`, …) — the CSP forbids inline scripts. Use `data-action="fnName"` with `act()`/`actOn()` from `dispatch.js`; the handler must be exposed on `window`. The `$this`/`$event` tokens in args resolve to the element/event. `scripts/check-no-inline-handlers.sh` enforces this in CI.
- Recent orders sync live via `onSnapshot` (see `loadOrders`); reports/customers still load full history on demand and set `fullOrdersLoaded` so the live listener won't clobber it

## Model routing

- **opus** — `firestore.rules`, `dispatch.js`/architecture changes,
  XSS/security-sensitive code (user content must go through `esc()`).
- **sonnet** — tools modules (`web/js/tools/*`), UI polish, label layout,
  tests, routine fixes.

## Loop protocol (unattended /loop sessions)

- Every iteration MUST end with `npm run check` passing before the next one
  starts; a failing check is the stop condition, not something to bypass.
- Deploy only via `./deploy.sh` (never raw `firebase deploy`).
- New JS module → add to the `SHELL` precache list in `web/sw.js`.
- Commit each green iteration; never stack uncommitted iterations.

---
> Source: [praneeth-goparaju/batterup](https://github.com/praneeth-goparaju/batterup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
