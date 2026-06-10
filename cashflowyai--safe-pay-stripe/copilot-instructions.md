## safe-pay-stripe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Safe Pay for Stripe** — a Manifest V3 Chrome extension that injects a widget into
`dashboard.stripe.com` showing how much the user can safely pay themselves this
month. It computes this from the user's own Stripe revenue, fetched via a
restricted (read-only) API key the user provides.

There is no build step, no package manager, and no test runner. Load it via
`chrome://extensions` → Developer mode → "Load unpacked" → this directory.
Reload from that page after any change. The service worker has its own DevTools
console (linked from the extensions page); content-script logs appear in the
Stripe dashboard tab's console.

## Architecture

Three scripts split by execution context — this split is the load-bearing design
decision, so respect it:

- **`lib/calc.js`** — pure, DOM-free, chrome-API-free math + date helpers
  (`computeSlices`, `trailingPeriod`, `formatMoney`, `clampPct`). Loaded in
  *both* the content script (as `window.SafePayCalc`) and the service worker (as
  `self.SafePayCalc`), and `module.exports` for node. Keep it free of any DOM or
  `chrome.*` reference so it stays shared and unit-testable.
- **`background.js`** (service worker) — owns all `api.stripe.com` calls. The
  Stripe key lives here and is sent via the `Authorization` header; it must
  **never** touch page context. `host_permissions` is what lets this worker call
  Stripe cross-origin without CORS. Communicates with the content script over
  `chrome.runtime` messages (`{type:"fetchRevenue", apiKey}` → `{ok, revenue,
  currency, period}`).
- **`content.js`** — the widget UI on the Stripe page. Holds all DOM/render
  logic and a single `state` object with a `status` machine
  (`loading | onboarding | ready | error`). Renders by setting `innerHTML` and
  re-wiring event listeners each `render()`.

`manifest.json` wires it together: `lib/calc.js` loads before `content.js`, both
scoped to `https://dashboard.stripe.com/*`.

## Things that will bite you

- **The Stripe dashboard is an obfuscated React SPA with no stable class names.**
  The widget anchors to the semantic `<main>` landmark, falling back to a fixed
  overlay. A `setInterval(ensureWidget, 1500)` re-anchors the node when React
  unmounts/navigates. Don't rely on Stripe DOM selectors beyond `main` /
  `[role="main"]`.
- **`render()` blows away DOM** (full `innerHTML` rewrite), so it's only called on
  first creation and on explicit state changes — not on every `ensureWidget`
  tick — to preserve drawer state and input focus. Moving the existing node
  doesn't re-render.
- **All CSS is scoped under `#safepay-widget`** in `content.css` so nothing leaks
  into (or in from) the Stripe dashboard. Keep new rules under that prefix.
- **All user HTML must go through `escapeHtml` / `escapeAttr`** — strings are
  concatenated into `innerHTML`.
- **API key validation** lives in two places (content `hasValidKey` and
  background `fetchRevenue`): keys must match `/^rk_/` — **restricted keys only**.
  Secret keys (`sk_`) are deliberately rejected so a user can't paste an
  all-powerful live key; onboarding steers users toward the read-only `rk_` key.
- The onboarding flow deep-links to Stripe's key-creation page with name and the
  `rak_charge_read` permission pre-selected (`createKeyUrl`). The `/test/` path
  variant is chosen when the user is on a test/sandbox dashboard. If
  `background.js` starts reading other Stripe resources, update the `perms` array
  there to match.

## Domain logic

`computeSlices` splits revenue into four slices that **sum to revenue** (so the
donut tiles a full ring with no base track):
`payYourself = (revenue − expenses) × (1 − tax% − buffer%)`, plus `expenses`,
`tax`, `buffer`. `trailingPeriod` defines the window as the **3 full calendar
months before the current month** (current month excluded), returning Stripe
`created[gte]/[lt]` unix-second bounds and display labels.

---
> Source: [Cashflowyai/safe-pay-stripe](https://github.com/Cashflowyai/safe-pay-stripe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
