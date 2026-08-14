## dashboard

> Static browser dashboard (HTML + vanilla JS + CSS). No build tool, no package manager, no tests.

# Agent Notes: Dashboard AC

## Project type
Static browser dashboard (HTML + vanilla JS + CSS). No build tool, no package manager, no tests.

## Run locally
- Open `index.html` directly in a browser.
- Or serve the folder with any static server:
  ```bash
  python -m http.server 8000
  # then open http://localhost:8000
  ```
- If using VS Code Live Server, `.vscode/settings.json` pins the port to `5501`.

## External dependencies
Loaded from CDN in `index.html`:
- `mqtt` (HiveMQ public broker via `wss://broker.hivemq.com:8884/mqtt`)
- `chart.js`
- Google Fonts

Do not assume these are installed locally.

## Backend & data
- Historical data comes from a Google Apps Script endpoint (`CONFIG.APPS_SCRIPT_URL` in `js/core/config.js`).
- Real-time data and commands use MQTT topics defined in `js/core/config.js`.
- If the dashboard appears blank or charts never load, check the browser network tab for Apps Script CORS/errors and MQTT connection failures first.

## Code architecture
### Core layer (`js/core/`) — pure logic, no DOM
- `js/core/config.js` — single source of truth: Apps Script URL, MQTT broker, topic names. Frozen with Object.freeze().
- `js/core/helpers.js` — pure functions: THI/COP calculation, date formatting, classification, constants.
- `js/core/mqtt.js` — MQTT client wrapper with exponential backoff reconnect, Promise-based publish(), connection status callbacks.
- `js/core/api.js` — fetch wrapper with error handling and user-facing notifications.

### UI layer (`js/ui/`) — DOM manipulation & Chart.js
- `js/ui/notification.js` — centralized toast notification system (replaces alert()).
- `js/ui/monitoring.js` — live metric card updates with cached DOM references and MQTT handlers.
- `js/ui/charts.js` — Chart.js lazy initialization (getOrCreate pattern). Monitoring + energy/TDL charts only.
- `js/ui/config-tab.js` — config tab UI handlers (AC mode, ESP mode, IR capture, WiFi — currently disabled).

### Entry point
- `js/app.js` — entry point: connects MQTT, binds handlers, event delegation, Page Visibility-aware polling.

## Important conventions
- Script load order in `index.html` matters (`core/config.js` → `core/helpers.js` → `core/mqtt.js` → `core/api.js` → `ui/notification.js` → `ui/monitoring.js` → `ui/charts.js` → `ui/config-tab.js` → `app.js`).
- All JS is IIFE-based, exposed via `window.Dashboard.*` (`window.Dashboard.MQTTClient`, `window.Dashboard.API`, etc.).
- Use the `CONFIG.TOPICS` map for any new MQTT topic; do not hard-code topic strings elsewhere.
- Event handling uses delegation via `data-*` attributes (e.g., `data-tab`, `data-ac-cmd`, `data-filter-energy`). No inline `onclick`/`onchange`.
- Keep UI text in Indonesian; the dashboard language is `id`.
- CSS split into `base.css` (variables, reset, grid, responsive) and `components.css` (all component styles). Semantic class names only (e.g., `.card-daya`, `.card-thi`).

## IR capture flow
- `CAPTURE` — publishes the selected button to start capturing.
- `CAPTURE_RESULT` — ESP publishes the raw IR code back here; `ConfigTab.bindCaptureResult()` displays it.
- `CAPTURE_CONFIRM` — confirms/saves the captured code.
- AC ON/OFF commands use a separate topic `AC_COMMAND` (not `CAPTURE_CONFIRM`).

## Security notes
- WiFi config feature is currently DISABLED (`sendWifiConfig` shows a notification only). Sending SSID/password via public MQTT broker needs a secure implementation (ESP AP mode).
- `CONFIG` and `CONFIG.TOPICS` are frozen at init time to prevent accidental mutation.

## What not to expect
- No lint, formatter, typecheck, or CI config.
- No `package.json`, lockfile, or framework router.
- No automated tests.

---
> Source: [anitaneza/dashboard](https://github.com/anitaneza/dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
