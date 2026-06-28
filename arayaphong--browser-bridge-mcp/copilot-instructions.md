## browser-bridge-mcp

> This file is written for AI coding agents. It assumes no prior knowledge of the project.

# Agent Guide: browser-bridge-mcp

This file is written for AI coding agents. It assumes no prior knowledge of the project.

## Shell convention

- Use `fish` as the user's shell for this project. Prefer fish-compatible commands and syntax when shell behavior matters.

## Project overview

`browser-bridge-mcp` is a local demo / integration harness for bidirectional Browser ↔ AI communication using Playwright and a Chrome extension. It provides:

- A simple Vite-served demo web page (`index.html`) with randomized system-status widgets, buttons that deliberately trigger console errors and 404 network requests, and an intentionally broken layout section. This is designed to produce interesting Playwright/MCP capture output.
- A Chrome extension that lets the user click any page element and send its selector and context to a local Node.js receiver.
- A Node.js selection receiver server (`scripts/selection-server.mjs`) that persists the picked element to `<outputDir>/<hostname>/last-selected-element.json` and archives past selections under `<outputDir>/<hostname>/history/`.
- A Browser Bridge MCP server (`scripts/bridge-mcp-server.mjs`) that exposes the latest selection and history to any MCP-compatible AI.
- A Playwright capture script (`scripts/capture-context.mjs`) that can screenshot and inspect either a URL or the last selected element.
- An MCP server configuration (`.vscode/mcp.json`) that wires Playwright and Browser Bridge MCP tools to the IDE/chat client.

The project is intentionally minimal: plain HTML/CSS/JS, no frontend framework, no test runner, no production build, and no CI/CD.

## Technology stack

- **Runtime / package manager:** Node.js, npm, ES Modules.
- **Web app dev server:** Vite 6 (default config; no `vite.config` file exists).
- **Frontend:** HTML5, CSS3, vanilla JavaScript.
- **Browser automation:** Playwright (Chromium).
- **MCP server:** `@executeautomation/playwright-mcp-server`.
- **Browser extension:** Chrome Manifest V3 with service worker, content script, and popup.
- **Local receiver:** Plain Node.js `http` server.

## Directory structure

```
/home/arme/Projects/browser-bridge-mcp
├── AGENTS.md                 # Agent instructions (this file)
├── BROWSER_BRIDGE.md         # Human-readable architecture & setup guide
├── docs/
│   └── BROWSER_BRIDGE_EXTENSION_USAGE.md  # How to use the extension with any MCP-compatible AI
├── package.json              # npm scripts and dependencies
├── package-lock.json
├── index.html                # Vite-served demo web app
├── main.js                   # Demo app logic
├── style.css                 # Demo styles
├── start-local.sh            # Start the Vite demo web app
├── start-bridge.sh           # Start the MCP + selection receiver servers
├── stop-bridge.sh            # Stop the bridge servers started by start-bridge.sh
├── .vscode/
│   ├── mcp.json              # MCP server definition (Playwright + Browser Bridge, stdio)
│   └── settings.json         # MCP access / discovery settings
├── extension/                # Unpacked Chrome extension
│   ├── manifest.json         # Manifest V3
│   ├── background.js         # Service worker icon-click handler
│   ├── content.js            # Element picker, selector generation, sender
│   ├── popup.html            # Extension popup markup
│   ├── popup.js              # Popup toggle/status logic
│   └── icons/                # icon16/32/48/128.png
├── scripts/
│   ├── selection-server.mjs  # HTTP receiver on port 8932
│   └── capture-context.mjs   # Playwright capture/reporting script
├── output/                   # Generated artifacts
│   ├── last-selected-element.json
│   ├── *-context.json
│   ├── *-screenshot.png
│   └── mcp-screenshots/
├── .pids/                    # PID files for background bridge servers
└── .logs/                    # Logs for background bridge servers
```

## Runtime architecture

The project is meant to run as several co-located processes plus a browser extension:

1. **Vite dev server** — `http://127.0.0.1:5173` — serves the demo web app.
2. **Playwright MCP server** — `BROWSER_BRIDGE.md` shows `http://localhost:8931/mcp` when started with `--port 8931`; `.vscode/mcp.json` uses stdio transport instead.
3. **Browser Bridge MCP server** — stdio MCP server (`scripts/bridge-mcp-server.mjs`) that exposes `bridge://latest`, `bridge://history`, and related tools.
4. **Selection receiver server** — `http://127.0.0.1:8932` — receives element selections from the extension.
5. **Chrome extension** — loaded unpacked from `extension/`, injects a content-script picker into any page.

Data flow:

- **AI → Browser:** An MCP client uses Playwright tools to navigate, click, screenshot, and inspect pages.
- **Browser → AI:** The extension captures a clicked element, generates a selector, and POSTs JSON to the selection server. The server writes `output/last-selected-element.json`, and `npm run capture:last-selected` can read it back with Playwright.

### Key runtime files

- **`index.html` / `main.js` / `style.css`** — Demo single-page app.
  - Random CPU/memory values update every 5 seconds.
  - `btn-fetch` shows mocked JSON in `#result-area`.
  - `btn-error` deliberately logs errors/warnings and throws a caught exception.
  - `btn-network` fetches `/api/status` (intentionally 404).
  - "Broken Layout Demo" uses `overflow: hidden` and a translated box to produce visual clipping.
  - Several elements have `data-testid` attributes for Playwright-friendly selectors.

- **`extension/manifest.json`** — Manifest V3 with `activeTab`, `scripting`, and `<all_urls>` host permissions.

- **`extension/background.js`** — Listens for toolbar icon clicks and sends a `toggle` message to the active tab.

- **`extension/popup.js` + `popup.html`** — Popup UI with Start/Stop selecting button; injects `content.js` if needed, queries selection status, and toggles mode.

- **`extension/content.js`** — Core picker logic:
  - Draws a hover overlay with a very high `z-index`.
  - Generates CSS selectors preferring `#id`, then `[data-testid]`, then a class/tag path with `:nth-of-type`.
  - Captures element metadata (`tagName`, `text`, `outerHTML`, `boundingBox`, etc.).
  - Captures a cropped screenshot of the picked element via the background service worker.
  - POSTs JSON (including the page URL, selector, HTML, text, and a base64 screenshot) to `http://localhost:8932/selected`.
  - Shows success/error toast notifications.

- **`scripts/selection-server.mjs`** — Plain Node.js `http` server.
  - `GET /health` — health check.
  - `GET /latest` — returns the most recent selection JSON across all projects.
  - `GET /selected` — returns the saved selection JSON for a project.
  - `GET /history?project=<id>` — returns all past selections for a project.
  - `POST /selected` — derives the project ID from the page URL hostname, then writes the selection to `<outputDir>/<projectId>/last-selected-element.json`, saves the screenshot to `<outputDir>/<projectId>/last-selected-element.png`, archives a snapshot under `<outputDir>/<projectId>/history/`, and also updates the global `<outputDir>/last-selected-element.json` + `.png`.
  - `POST /clear` — clears the latest selection, screenshot, and history for a project.
  - CORS headers mirror the request origin or fallback to `*`.

- **`scripts/capture-context.mjs`** — Playwright (Chromium, headless) script.
  - Navigates to a URL (default `https://example.com`, or from `--use-last-selected`).
  - Optionally locates a selector from CLI `--selector` or the last selection.
  - Collects console logs, network responses, page errors, and request failures.
  - Saves a full-page or element screenshot plus a JSON context report to `output/`.

## Build, development, and test commands

From `package.json`:

| Command | Purpose |
|---|---|
| `npm run dev` | Start the Vite dev server at `http://127.0.0.1:5173`. |
| `npm run selection-server` | Start the selection receiver on `http://127.0.0.1:8932`. |
| `npm run capture` | Run the Playwright capture script. |
| `npm run capture:last-selected` | Run the capture script using `output/last-selected-element.json`. |
| `npm run mcp:playwright` | Launch the Playwright MCP server via npx. |
| `npm run mcp:bridge` | Launch the Browser Bridge MCP server (stdio). |

Convenience scripts for persistent local use:

| Script | Purpose |
|---|---|
| `./start-local.sh` | Start the Vite dev server for the demo web app. |
| `./start-bridge.sh` | Start both the Playwright MCP HTTP server (port `8931`) and the selection receiver (port `8932`) in the background, and run health checks. |
| `./stop-bridge.sh` | Stop the bridge servers started by `start-bridge.sh`. |

Useful manual commands (from `BROWSER_BRIDGE.md`):

```bash
# Playwright MCP HTTP server on port 8931
CHROME_EXECUTABLE_PATH=/usr/bin/google-chrome-stable npx -y @executeautomation/playwright-mcp-server --port 8931

# Capture a specific selector manually
node scripts/capture-context.mjs http://127.0.0.1:5173/ --selector '[data-testid="disk-status"]'
```

Important notes:

- There are no `build`, `test`, `lint`, or `format` scripts.
- There is no automated test suite.
- Vite has no custom configuration file; it relies entirely on defaults.

## Code organization and main modules

| Module | Responsibility |
|---|---|
| `index.html` | Demo page markup. |
| `main.js` | Demo app logic: random status updates, button handlers, intentional errors. |
| `style.css` | Dark-themed demo styles, including intentionally broken layout demo. |
| `extension/manifest.json` | Extension packaging and permissions. |
| `extension/background.js` | Toolbar-icon click broker. |
| `extension/popup.js` / `popup.html` | User-facing extension controls. |
| `extension/content.js` | In-page element picker, selector generation, overlay, and sender. |
| `scripts/selection-server.mjs` | Receive and persist element selections. |
| `scripts/bridge-mcp-server.mjs` | MCP server exposing selections to AI assistants. |
| `scripts/capture-context.mjs` | Playwright-based capture and reporting. |
| `.vscode/mcp.json` | IDE MCP integration config. |
| `.vscode/settings.json` | MCP access/discovery settings. |
| `output/` | Generated artifacts (screenshots, context JSON, last selection). |
| `start-local.sh` | Start the Vite demo web app. |
| `start-bridge.sh` | Start MCP + selection servers for a session. |
| `stop-bridge.sh` | Stop servers started by `start-bridge.sh`. |

## Code style guidelines

- **Language:** Plain JavaScript; ES2022+ features such as top-level `await` and optional chaining are used.
- **Modules:** Node scripts use `.mjs` with ESM `import`; use `import.meta.url` to derive `__dirname`.
- **Indentation:** 2 spaces.
- **Quotes:** Single quotes in JavaScript; single quotes in CSS font stacks where applicable.
- **Semicolons:** Used consistently.
- **Trailing commas:** Used in multi-line objects and arrays.
- **Naming:**
  - `camelCase` for variables and functions.
  - `kebab-case` for IDs and CSS classes.
  - `UPPER_SNAKE_CASE` for module-level constants such as `SERVER_URL` and `OUT_DIR`.
- **CSS:** Custom properties in `:root`, mobile media query, utility-style class names.
- **Comments:** Sparse; comments explicitly mark intentional demo/broken behavior.
- **Inline styles:** Heavy use in `extension/content.js` for dynamically injected overlay/toast DOM.

## Testing instructions

- There is no automated test suite, test runner, or assertions.
- Manual QA workflow:
  1. `npm run dev` (or `./start-local.sh`) to start the web app.
  2. `npm run selection-server` (or `./start-bridge.sh`) to start the receiver.
  3. Load `extension/` as an unpacked Chrome extension.
  4. Open `http://127.0.0.1:5173/`, click the extension icon, and select an element.
  5. Run `npm run capture:last-selected` and verify the generated `output/*-context.json` and screenshots.

## Security considerations

- The extension requests broad permissions: `activeTab`, `scripting`, and `<all_urls>` host permission.
- The selection server binds to `127.0.0.1` (limiting remote exposure) but has no authentication and mirrors the request origin in CORS headers, allowing any local page to POST.
- There is no input schema validation beyond `JSON.parse`.
- Playwright launches the browser binary pointed to by `CHROME_EXECUTABLE_PATH` (defaults to `/usr/bin/google-chrome-stable`) in headless mode with default arguments.
- This project is a local demo/trust-boundary tool; do not expose the selection server or MCP server on a public network.

## Deployment and release

- There is no CI/CD pipeline, Dockerfile, or deployment script.
- The directory is not a git repository.
- The browser extension is loaded manually in Chrome via **Developer mode → Load unpacked**, pointing at the `extension/` folder.
- The web app runs locally via `npm run dev`.
- No production bundle or release artifact is produced.

## Common troubleshooting

- **Extension says it cannot connect?** Ensure `npm run selection-server` is running on port `8932`.
- **Selection not saving?** Check the browser console for CORS or network errors, and verify the server is reachable at `http://127.0.0.1:8932/health`.
- **Capture script cannot find the element?** The page may have changed since selection; refresh and re-select.
- **`capture-context.mjs` fails to launch the browser?** Set the `CHROME_EXECUTABLE_PATH` environment variable to your Chrome/Chromium binary.

## Environment variables

| Variable | Default | Used by |
|---|---|---|
| `CHROME_EXECUTABLE_PATH` | `/usr/bin/google-chrome-stable` | Playwright MCP server, `scripts/capture-context.mjs` |
| `SELECTION_SERVER_PORT` | `8932` | `scripts/selection-server.mjs` |
| `SELECTION_OUTPUT_DIR` | `/tmp/browser-bridge-output` | `scripts/selection-server.mjs`, `scripts/bridge-mcp-server.mjs` |

---
> Source: [arayaphong/browser-bridge-mcp](https://github.com/arayaphong/browser-bridge-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
