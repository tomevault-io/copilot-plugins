## sighthound-savings-analyzer

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project overview

- This is a small, static web app that implements the **Sighthound Savings Analyzer**: a multi-step wizard that collects camera and cost inputs, then computes an optimal Sighthound configuration and savings.
- The core runtime lives entirely in the browser: `index.html` (markup), `styles/savings-analyzer.css` (layout and theming), and `app.js` (behavior and calculations).
- There is **no bundler or framework** in use (no React/Vue/etc.); JavaScript runs as a single module script attached directly to `index.html`.
- A Node-based harness `test-run.js` uses `jsdom` to simulate the browser and exercise the wizard flow for regression testing.

The root `README.md` only names the project; there are no additional build or workflow details defined there.

## Common commands

All commands assume the project root as the working directory.

### Setup

- Install Node dependencies (only `jsdom` plus transitive deps):
  - `npm install`

### Running the app locally

There is no dedicated dev server or build step; the app is a static HTML/CSS/JS bundle.

- Quick local preview (using any static file server):
  - `npx serve .`
  - Then open the reported URL (typically `http://localhost:3000`) and navigate to `/index.html`.
- Alternatively, open `index.html` directly in a browser; prefer using an HTTP server if you run into module/CORS restrictions with `file://` URLs.

### Tests / harness

There is **no formal test runner configured in `package.json`**; the default `npm test` script just exits with an error placeholder. Instead, use the Node harness:

- Run the jsdom-based navigation harness:
  - `node test-run.js`
- This script will:
  - Load `index.html` into `jsdom`.
  - Wait for the app to signal that initialization completed via `window.__savings_init_done` (set in `app.js`).
  - Simulate typical user interactions (choosing a camera option, proceeding through steps, skipping step 3, navigating back and forth, toggling software selection) and log the active step IDs and button state to stdout.
- To focus on a specific scenario, temporarily comment out sections of `test-run.js` and re-run `node test-run.js`; there is no granular "single test" mechanism beyond editing this harness.

## High-level architecture

### Files and responsibilities

- `index.html`
  - Defines the multi-step wizard structure as five `.step` sections (`#step1`, `#step1b`, `#step2`, `#step3`, `#step4`, `#step5`) plus a `#results` section.
  - Uses semantic IDs (e.g., `continueStep2`, `backStep3`, `totalCamerasDisplay`, `setupGrid`, `costComparison`, `savingsCard`, `downloadPdf`) that are hard-wired into `app.js`.
  - Attaches the main behavior via `<script type="module" src="./app.js"></script>` and pulls in `styles/savings-analyzer.css`.

- `styles/savings-analyzer.css`
  - Contains a full, pre-generated stylesheet (Webflow-style reset + Sighthound-branded theme) and all layout/typography for the analyzer.
  - Defines visual states such as `.step.active`, `.node-status` variants, `.timeframe-btn.active`, and button styles used by `index.html`.
  - There is **no build pipeline** for CSS; this is the final stylesheet served to the browser.

- `app.js`
  - Owns **all interactive behavior and business logic** for the wizard and results, structured around a single mutable `state` object and a set of helper functions.
  - Attaches event listeners once (via `init()`), but uses some **delegated handlers** (notably a body-level click listener) to remain robust in different environments (browser vs. jsdom).
  - Exposes a `window.__savings_init_done` flag so external harnesses (like `test-run.js`) can detect when initialization has completed.

- `test-run.js`
  - Headless integration harness built on `jsdom`.
  - Loads `index.html`, applies workarounds for canvas and image APIs used by jsPDF, waits for the app to initialize, and then programmatically drives the UI.
  - Uses real `MouseEvent` and `change` events to stay close to real browser behavior.

### State and navigation model (`app.js`)

- **Global state**
  - `state` is a plain object holding user selections and derived settings: current step, camera type, ownership model, counts for standard/smart cameras, compute nodes, auto-add toggle, selected software options, current costs, billing frequency, and comparison timeframe.
  - This state is the single source of truth for both UI updates and calculations; all helper functions read/write from it.

- **Initialization lifecycle**
  - `init()` is idempotent (guarded by an internal `__savings_init_done` boolean) and can run safely even if triggered multiple times from different DOM lifecycle events.
  - Initialization sequence:
    - Attach all event handlers.
    - Derive initial cameras/nodes and selected software state.
    - Initialize the continue-button state for step 3.
    - Navigate to step 1.
    - Finally, set `window.__savings_init_done = true` to signal readiness to external tools.
  - `init()` is wired to all of: `DOMContentLoaded`, `window.load`, and a `setTimeout(init, 0)` if the document is already past `loading`, ensuring it runs in both real browsers and jsdom.

- **Step navigation**
  - `goToStep(step)` controls which `.step` section is active, updates the progress bar, and tracks the current step in `state.step`.
  - `step` is usually a number `1`–`5` but may also be the string `'1b'` for the conditional ownership step; `goToStep` normalizes this when updating the progress indicator.
  - Progress text is derived from a constant `totalSteps = 5`; if you add or remove steps, you must update this constant and the DOM structure in `index.html` together.
  - All navigation buttons (`continueStepX`, `backStepX`, `skipStep3`, `startAnalysis`, `editAnswers`, `startOver`) are wired through click handlers that **prevent default** browser behavior before calling `goToStep` or higher-level actions (like `runAnalysis()`).

### Camera and node capacity logic

- Cameras
  - Two camera categories: standard IP cameras and Sighthound smart cameras, each with individual prices from the `PRICES` constant.
  - Counts are controlled via **stepper buttons** (`data-target="standardCameras"` / `"smartCameras"`) and mirrored `<input type="number">` fields; both update the shared `state` and call `updateCamerasAndNodes()`.

- Compute nodes and auto-add
  - `CAMERAS_PER_NODE` defines capacity per compute node (currently 4 cameras per node).
  - `updateCamerasAndNodes()`:
    - Calculates the total camera count and a suggested node count based on `Math.ceil(totalCameras / CAMERAS_PER_NODE)`.
    - When `state.autoAddNodes` is `true`, automatically syncs `state.computeNodes` and the `#computeNodes` input to this suggestion, disables manual editing, and disables the node stepper buttons.
    - When `autoAddNodes` is `false`, re-enables manual editing and stepper buttons.
    - Delegates to `updateNodeStatus(totalCameras, suggestedNodes)` to populate the `#nodeStatus` banner.
  - `updateNodeStatus` computes capacity vs. demand and sets both the message and CSS class:
    - `neutral` when there are zero cameras.
    - `info` with a suggested node count if zero nodes are selected.
    - `warning` when capacity is insufficient for configured cameras.
    - `success` when capacity covers or exceeds configured cameras.

### Software selection and optional step 3

- Step 3 presents optional software analytics; checkboxes have `name="software"` and `data-price` attributes.
- `updateSelectedSoftware()` collects all checked options and stores them in `state.software` as `{ type, price }` objects, where `price` is the per-stream monthly cost.
- `updateContinueStep3State()` simply enables or disables the `#continueStep3` button based on whether `state.software` is non-empty.
- Two important flows:
  - Normal: user selects at least one software checkbox; `continueStep3` becomes enabled and moves to step 4.
  - Optional: user clicks `skipStep3`, which explicitly clears all selected software, updates state/continue-button state, and jumps straight to step 4. Returning to step 3 after a skip should still present an empty selection and a disabled `continueStep3`.
- The jsdom harness in `test-run.js` exercises both flows and back-navigation between steps 3 and 4; if you change IDs or behavior here, you must update the harness accordingly.

### Cost and savings computation

- Core computations are centralized but reused across several functions:
  - `runAnalysis()` computes current totals (using helpers) and then:
    - Calls `updateRecommendedSetup(monthlySoftwareTotal)` to populate the hardware/software breakdown grid.
    - Calls `updateCostComparison()` to update the side-by-side cost comparison.
    - Calls `updateSavingsCard()` to show high-level savings or additional investment.
    - Finally, reveals the `#results` section and scrolls it into view.

- `updateRecommendedSetup(monthlySoftwareTotal)`
  - Computes hardware costs for standard cameras, smart cameras, and compute nodes using `PRICES` and the current `state`.
  - Builds human-readable descriptions of each hardware component (including camera counts and node capacity) and a line summarizing monthly software spend per selected analytics type.
  - Renders all of this into `#setupGrid` using `innerHTML` and a shared currency formatter (`Intl.NumberFormat`).

- `updateCostComparison()`
  - Uses the same hardware and software totals alongside user-provided current costs from `#currentMonthly` / `#currentUpfront`.
  - Normalizes current monthly cost if the user bills annually (by dividing by 12) before multiplying by `state.timeframe`.
  - Renders a two-column comparison in `#costComparison`, showing both the current setup and the Sighthound configuration for the chosen timeframe.

- `updateSavingsCard()`
  - Computes `savings = currentTotal - sighthoundTotal` and `savingsPerMonth = savings / state.timeframe`.
  - Updates the `#savingsCard` element:
    - When `savings > 0`, shows expected savings over the timeframe and per-month savings and uses the default ("positive") styling.
    - When `savings <= 0`, switches to a `neutral` style and explains that there is additional investment, framing it as upgraded hardware + analytics.

- `generatePDF()`
  - Lazily loads jsPDF from a CDN if not already present, handling both `window.jspdf.jsPDF` and `window.jsPDF` exports.
  - Recomputes the same hardware, software, and cost totals used in the UI to ensure the PDF matches on-screen values.
  - Writes a concise summary of configuration, selected software, and cost breakdown to a single-page PDF and triggers a download as `savings-analysis.pdf`.
  - If you change the pricing model or hardware assumptions, update the shared constants and ensure `generatePDF()` remains in sync with the UI helpers.

### Timeframe selection

- Timeframe buttons in the results section (`.timeframe-btn` with `data-months`) control `state.timeframe` and the active styling.
- A click handler:
  - Clears the `.active` class from all timeframe buttons and sets it on the clicked one.
  - Updates `state.timeframe` from the button's `data-months`.
  - Re-runs `updateCostComparison()` and `updateSavingsCard()` to reflect the new horizon.

## Testing harness details (`test-run.js`)

- Uses `JSDOM.fromFile('index.html', { runScripts: 'dangerously', resources: 'usable', url: 'http://localhost:8000/' })` to approximate a real browser environment.
- Installs a **canvas shim** and a minimal `Image` implementation in `beforeParse` to avoid `jsdom` throwing on canvas access (required for jsPDF compatibility).
- Waits for `window.load`, then (if needed) busy-waits up to a max timeout for `window.__savings_init_done` to become truthy.
- Retrieves key navigation buttons by ID and logs their presence and disabled state, then simulates interactions:
  - Select a step 1 option (`data-value="none"`) and verify that the active step changes.
  - Click `continueStep2` and assert that the active step becomes step 3.
  - Exercise the `skipStep3` path to jump to step 4, then go back to step 3 and verify that `continueStep3` remains disabled with no software selected.
  - Select a software option, ensure `continueStep3` becomes enabled, click it to proceed to step 4, then go back again.
- The script exits with code `0` on success and `2` on failure. When evolving navigation behavior, keep this file in sync or expand it to cover new flows.

## Warping/AI meta-guidelines in this repo

- The `warping/` directory vendors in the **Warping framework** documentation used across multiple projects. It defines general AI behavior, language/tool guidelines, and rule precedence, but it is **not specific to this savings analyzer**.
- Key references for future agents, if you need broader guidance:
  - `warping/main.md` – overall AI behavior and rule-precedence model.
  - `warping/core/project.md` – guidelines for the Warping framework project itself (largely about Markdown documentation, not this JS app).
- When instructions in `AGENTS.md` and `warping/` conflict for this repository, treat **this `AGENTS.md` file as the source of truth for working with the savings analyzer code.

---
> Source: [sighthoundinc/sighthound-savings-analyzer](https://github.com/sighthoundinc/sighthound-savings-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
