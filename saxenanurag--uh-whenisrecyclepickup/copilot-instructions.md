## uh-whenisrecyclepickup

> This is a simple, static web application designed to help residents of University Heights determine their recycling pickup schedule for the year 2026. The app takes a street name as input and calculates the next pickup date based on the city's "Route A" and "Route B" bi-weekly schedule.

# Project Context: University Heights Recycling Schedule

## Project Overview

This is a simple, static web application designed to help residents of University Heights determine their recycling pickup schedule for the year 2026. The app takes a street name as input and calculates the next pickup date based on the city's "Route A" and "Route B" bi-weekly schedule.

### Key Technologies
*   **Frontend:** Vanilla HTML5, CSS3, and JavaScript (ES6+).
*   **Data Storage:** Hardcoded JavaScript arrays in `data.js` (acting as a lightweight database).
*   **No Backend:** The application runs entirely in the client's browser.

## Directory Structure

*   `index.html`: The main entry point and user interface.
*   `script.js`: Core logic for search, date calculation, and DOM manipulation.
*   `data.js`: Contains `STREET_DATA` (street-to-route mappings), `HOLIDAYS_2026`, and `A_ROUTE_START` reference date.
*   `styles.css`: Styling for the application.
*   `tests/`: Contains automated tests.
    *   `test_search_logic.js`: Node.js script to verify search functionality (handles street suffix stripping).
    *   `verify_global_alert.py`: Python/Playwright script for visual verification of holiday alerts.

## Building and Running

Since this is a static site, there is no compile step.

### Running Locally
You can simply open `index.html` in a web browser, or use a simple HTTP server for a better experience:

```bash
# Python 3
python3 -m http.server 8080
# Access at http://localhost:8080
```

### Testing
**1. Logic Tests (Node.js)**
Run the search logic tests (which mock the browser environment):
```bash
node tests/test_search_logic.js
```

**2. Visual/End-to-End Tests (Python/Playwright)**
*Requires `playwright` python package.*
```bash
python3 tests/verify_global_alert.py
```

**3. Manual Simulation Mode**
The app includes a hidden simulation mode for manual testing in the browser:
1. Open `index.html`.
2. Open Developer Tools Console.
3. Run `enableTesting()`.
4. Use the UI to simulate "Today" as any date in 2026.

## Development Conventions

*   **Data Integrity:** Modifications to street data or holidays should be made in `data.js`. Ensure the structure of `STREET_DATA` objects remains consistent (`{ name, route, day, segment? }`).
*   **Search Logic:** The `findStreet` function in `script.js` handles query normalization (stripping suffixes like "Rd", "Ave").
*   **Date Handling:** All date logic is relative to `A_ROUTE_START` (Jan 5, 2026). Holiday delays are calculated dynamically based on the specific week.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saxenanurag)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/saxenanurag)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
