## creditcard-offers-activator

> 💳 Bank Offer Bookmarklet Factory: Dev Blueprint

💳 Bank Offer Bookmarklet Factory: Dev Blueprint

🎯 Project Goal
Build and maintain a suite of JavaScript bookmarklets that automate "Activate All" functionality for credit card merchant offers using a Python + Playwright + Gemini-CLI feedback loop.

🛠️ The 2026 Tech Stack
|Layer | Tool | Purpose |
|Automation Engine | Playwright (Python) | Navigation, Shadow DOM piercing, and session persistence.|
|AI Intelligence | Gemini-CLI | DOM analysis, selector repair, and logic refactoring.|
|Logic Layer | Python 3.11+ | Orchestrates browser, file-watching, and AI prompting.|
|Script Format | IIFE (JavaScript) | Ensures isolation within bank portals.|

🏆 Chase v4.5 Feature Set (Current Stable)
The Chase activator has evolved into a robust "Smart Activator" with the following capabilities:
- **AI Smart Learning Mode:** Allows the user to "teach" the script by clicking 2 sample buttons. It then learns the CSS selector and card container pattern dynamically.
- **Turbo DOM Scraper:** A hybrid speed approach that scans the card face for expiration info first, only waiting for the side-drawer if data is missing.
- **Date Calculation Engine:** Converts relative Chase terms like "3d left", "Last day", and "Expiring soon" into standardized `YYYY-MM-DD` formats based on the system date.
- **Navigation Interceptor:** Overrides `history.pushState` during activation to prevent the Single Page App (SPA) from redirecting the user away from the offers list.
- **Interactive HUD:** A dark-themed floating UI that displays real-time progress, success/fail counters, and provides mode-selection buttons.
- **CSV Export:** Generates a detailed report with Merchant Name, Expiration Date (formatted), Offer Details, and Activation Method.

🏆 Amex v3.7 Feature Set (Current Stable)
The Amex activator uses a "Structural Precision" strategy to handle complex nested DOM elements:
- **Smart Trait-Based Learning (3-Click Triangulation):** User clicks 3 different offers to train the engine. It triangulates shared Aria labels, titles, and framework-specific CSS classes.
- **Intelligent Ancestry Engine:** Scans the DOM tree up from the clicked button to identify the reliable repeating card container (using class scoring).
- **Double-Pass Scrape:** Captures expiration data both before clicking and 800ms after clicking (when Amex often reveals countdowns like "3d left").
- **Integrated Autoscroll:** Automatically handles lazy-loading by scrolling the page to ensure all offers are injected into the DOM before scanning begins.
- **Date Calculation Engine:** Same as Chase, standardizing all terms to `YYYY-MM-DD`.
- **API Deprioritization:** Reverted to a high-speed "Turbo DOM" approach due to Amex's aggressive CORS blocking and complex telemetry headers (ruxitagentjs).
- **CSV Export:** Unified schema matching the Chase version for cross-bank reporting.

🏆 Citi v2.4 Feature Set (Current Stable)
The Citi activator combines a high-speed API layer with a resilient "Ghost Stealth" DOM fallback:
- **API-First Bulk Activation:** Directly calls Citi's gcgapi endpoints to enroll all available offers across multiple cards in seconds.
- **Session Header Auto-Extraction:** Dynamically parses essential headers (TMXSessionId, client_id, etc.) from active session cookies.
- **Ghost Stealth Mode:** For DOM fallback, suppresses all CSS transitions and uses a MutationObserver to instantly nuke Citi's aggressive modal overlays.
- **Smart 3-Click Triangulation:** Uses a 3-sample training mode to precisely identify card containers and activation buttons.
- **Multi-Card CSV Reporting:** Automatically splits reports into separate files per physical card for easier reconciliation.

🚦 Differentiation StrategyTo manage multiple banks (Chase, Amex, Citi) without code collisions, we use a Strategy Pattern.
1. The "Universal Loader" (JS Level)Instead of five bookmarks, use one "Router" bookmarklet that detects the domain and fetches the correct logic.
    - How: Checks window.location.hostname.
    - Benefit: One click regardless of which bank portal you are currently viewing.

2. The "Bank Key" Mapping (Python Level)
The Python orchestrator treats each bank as a unique configuration object.
```# Python # Internal Logic Example
BANKS = {
    "amex": {"url": "amex.com/offers", "auth": "auth/amex.json", "src": "src/amex.js"},
    "chase": {"url": "chase.com/offers", "auth": "auth/chase.json", "src": "src/chase.js"},
    "citi": {"url": "citi.com/merchantoffers", "auth": "auth/citi.json", "src": "src/citi.js"}
}
```
3. The "Gemini Context" (AI Level)
When prompting Gemini-CLI for fixes, always prefix with the Bank Key.Prompt: "Working on the [AMEX] strategy. The selector #offer-stack is missing. Here is the HTML..."

📊 Data Standards: Universal CSV Schema
To ensure cross-bank compatibility and easy data merging, all bank portal activators must export CSVs following this standardized 7-column schema. This applies to all current and future implementations:

| Column | Requirement | Description | Example |
| :--- | :--- | :--- | :--- |
| **Card Kind** | Optional | The product name or nickname of the card. | `"Citi Double Cash"` |
| **Card Suffix** | Optional | The last 4 digits of the card account. | `"1234"` |
| **Merchant Name** | Mandatory | The name of the merchant offering the deal. | `"Starbucks"` |
| **Expiration Date** | Mandatory | The standardized expiration (YYYY-MM-DD). | `"2026-05-31"` |
| **Details** | Mandatory | The reward/offer description. | `"10% Back"` |
| **Method** | Mandatory | The activation technique used. | `"API (POST)"` or `"Turbo DOM"` |
| **Status** | Mandatory | Final outcome of the attempt. | `"Success"` or `"Failed"` |

*Note: If optional card fields are not available (common in DOM-only scrapers or legacy portals), use "N/A" as a placeholder to maintain column alignment and ensure files can be merged without error.*

🔄 The Development "Loop"
1. Persistent Auth: Log in manually once; save state via `context.storage_state(path="auth/bank.json")`.
2. Hot-Reload Injection: Python monitors /src for changes. On save, Playwright re-injects the specific bank script into the active tab.
3. Outside Evaluation: If the script fails, Python captures the DOM + Error. Gemini-CLI analyzes it and writes a patch directly back to the /src file.

📂 Project StructurePlaintext.
├── old_code/             # previous version old bookmarket
├── card_pointer_chrome_extension_copy/ a reference to market leader browser extension (cardPointerX)
├── auth/                 # Session JSONs (cookies/tokens per bank)
├── src/                  # Bank-specific logic (amex.js, chase.js)
├── dist/                 # Minified/Universal router bookmarklet
├── dev_watcher.py        # The Python "Brain" (Watcher + Playwright)
├── requirements.txt      # playwright, watchdog, jsmin
└── GEMINI.md             # This file

💡 Key Implementation Details
The "IIFE" Wrapper
Prevents variable leakage. Gemini-CLI should be instructed to always wrap output in this:

```// JavaScript
javascript:(function(){
    /* Bank-specific logic */
})();
```
Shadow DOM PiercingModern portals use Web Components. Playwright handles this natively (e.g., page.click('button') will find it inside a shadow root), but your bookmarklet JS may need specialized "piercing" functions. Use Gemini to generate these helpers.

# 💳 Bank Offer Bookmarklet Factory

## 🚀 Quick Start (uv)
To spin up the environment on a new machine:
1. **Sync Environment:** `uv sync`
2. **Install Browsers:** `uv run playwright install chromium`
3. **Launch Dev Watcher:** `uv run python dev_watcher.py`

## 🛠️ Environment Management
| Task | Command |
| :--- | :--- |
| **Add New Tool** | `uv add <package>` |
| **Run Scripts** | `uv run <script.py>` |
| **Update Lockfile** | `uv lock --upgrade` |
| **Clean Env** | `rm -rf .venv && uv sync` |

## 📂 Project Logic
- **Auth:** Keep session JSONs in `/auth` (Gitignored).
- **Source:** Write clean JS in `/src`.
- **Logic:** `dev_watcher.py` handles the file-watching and Playwright injection.

---
> Source: [leonchen417/creditcard_offers_activator](https://github.com/leonchen417/creditcard_offers_activator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
