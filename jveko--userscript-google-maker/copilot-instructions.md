## userscript-google-maker

> Modular Tampermonkey userscript that automates Google account signup flow with SMS verification. Source is split into ES modules under `src/`, bundled by esbuild into a single IIFE userscript for deployment. Targets iOS Safari/Userscripts app.

# AGENTS.md

## Project Overview
Modular Tampermonkey userscript that automates Google account signup flow with SMS verification. Source is split into ES modules under `src/`, bundled by esbuild into a single IIFE userscript for deployment. Targets iOS Safari/Userscripts app.

## Architecture
- **Modular source** (`src/`) → esbuild bundles → `dist/google-accounts.user.js`
- **Cloudflare Worker API** (`api-google-maker.cloudflare-5e0.workers.dev`) for profile generation, SMS handling, and account confirmation (with 35s native timeout failsafes)
- **Session persistence** via `GM_setValue`/`GM_getValue` (cross-origin, persists across `accounts.google.com` ↔ `myaccount.google.com`)
- **URL polling** (`setInterval`) detects page changes and dispatches to async page handlers
- **Human-like simulation** layer (fast randomized typing with typos, delays, dynamic scrolling containers, iOS native TouchEvent/PointerEvent simulation, Shadow DOM composed events, React native setters) for advanced anti-detection.
- **Stay iOS Extension Compatible** (Uses `setInterval` polling instead of `onurlchange`, plain parsed `GM_xmlhttpRequest`)

## Source Structure
```
src/
  main.js              — entry point, route table, error bubbling, init logic
  constants.js          — MONTH_NAMES, API_BASE, DELAY (optimized for speed), STATE, etc.
  state.js              — mutable shared state (config, currentState, lastErrorMsg, smsPoller)
  log.js                — DOM log panel + log() function
  session.js            — GM_setValue/GM_getValue persistence
  api.js                — apiRequest (with Failsafe timer), apiRequestWithRetry, fetchConfig
  human.js              — simulateMobileTouch, humanDelay, humanType, humanScroll, etc. (uses React native setter bypass)
  helpers.js            — waitFor (MutationObserver + visibility), awaitNavigationOrError, getElementByXpath
  sms.js                — SMS poller management
  handlers/
    signin.js, name.js, birthday.js, username.js, password.js,
    phone.js, sms-code.js, recovery.js, confirmation.js, terms.js,
    myaccount.js
  ui/
    panel.js            — floating Draggable control panel (Start/Stop/Reset, Status Monitor, Version Badge)
  banner.txt            — userscript header template (`{{VERSION}}` is replaced by esbuild)
```

## Code Style
- ES modules in `src/`, bundled into single IIFE by esbuild
- `GM_xmlhttpRequest` for cross-origin API calls (Stay native compatible)
- Async page handlers using `waitFor()` (which ensures element visibility) + `humanType()`/`humanDelay()` helpers
- XPath (`getElementByXpath`) for buttons, CSS selectors for inputs
- Submit & navigation logic uses `awaitNavigationOrError()` to bypass single-page inline validation without fixed delays
- Logging via custom `log()` that writes to both `console.log` and an on-page panel
- No TypeScript, no linter, no formatter — keep it plain ES2017+ JS
- Follow existing patterns: `async function handleXxxPage()` naming, `await humanDelay()` between actions

## Versioning & Deployment
- **Version Source:** Defined in `package.json` (bump using `npm version patch`)
- **Header Injection:** Esbuild automatically replaces `{{VERSION}}` in `src/banner.txt` during build
- **Build:** `npm run build` (esbuild bundles `src/main.js` → `dist/google-accounts.user.js`)
- **Watch:** `npm run watch` (rebuilds on file changes)
- **Deploy:** `./deploy.sh` (builds + deploys to Cloudflare Pages)
- **Install URL:** `https://google-maker.pages.dev/google-accounts.user.js`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jveko)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jveko)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
