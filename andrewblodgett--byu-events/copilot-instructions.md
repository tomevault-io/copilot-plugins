## byu-events

> Build a static, single-page web application (SPA) hosted on GitHub Pages that aggregates and displays upcoming BYU events. The application must fetch data client-side from the BYU Calendar API, render it into a clean, accessible interface, and provide sorting and filtering capabilities.

# Project Overview
Build a static, single-page web application (SPA) hosted on GitHub Pages that aggregates and displays upcoming BYU events. The application must fetch data client-side from the BYU Calendar API, render it into a clean, accessible interface, and provide sorting and filtering capabilities.

## Tech Stack & Design System
* **Core:** HTML5, CSS3 (Vanilla, CSS Variables), JavaScript (ES6+ Vanilla). No frameworks (e.g., React, Vue) to ensure easy GitHub Pages deployment.
* **Aesthetic:** Minimalist modern dark mode (Backgrounds: `#121212`, `#1e1e1e`; Text: `#e0e0e0`, `#b3b3b3`; Accents: subtle blue/purple).
* **Typography:** Clean sans-serif (Inter, system-ui).
* **Layout:** CSS Grid for event cards, Flexbox for control bars.

## API Integration Rules
* **Endpoint:** `https://calendar.byu.edu/api/Events.json`
* **Required Parameters:** `categories=all` (Failing to include this returns a 400 Bad Request).
* **Date Filtering:** Dynamically inject the current local date into `event[min][date]` (Format: `YYYY-MM-DD`).

---

## Execution Steps: Step-by-Step Implementation & Testing

Please implement this project strictly following these discrete phases. Do not move to the next phase until the current phase's verification criteria are met.

### Phase 1: Project Setup & HTML Shell
* **Action:** Create `index.html`. Set up the semantic HTML structure (`<header>`, `<main id="app">`, `<footer>`). Include standard meta tags for viewport and character set. Link an empty `style.css` and `app.js` (with `defer`).
* **Verification:** Open `index.html` in a browser. Open the browser console to ensure there are no 404 errors for the CSS or JS files.

### Phase 2: UI Mockup & Dark Theme CSS
* **Action:** In `style.css`, define CSS variables for the dark theme colors, spacing, and typography in the `:root` pseudo-class. Create the CSS for a responsive grid layout. Hardcode *one* dummy event card in the `index.html` to style it.
* **Verification:** The page should display a dark background with a well-formatted, readable placeholder event card. The card should be responsive (stacking on mobile, grid on desktop).

### Phase 3: API Service & Data Fetching
* **Action:** In `app.js`, write an `async` function `fetchEvents()`. Generate today's date in `YYYY-MM-DD` format. Construct the fetch URL using URLSearchParams to safely encode `categories=all` and `event[min][date]`. 
* **Verification:** Call `fetchEvents()` on page load. Use `console.log(data)` to print the JSON response. Inspect the browser console to confirm an array of event objects is successfully returned without 400/500 HTTP errors.

### Phase 4: Data Processing & State Management
* **Action:** Create a global or modular state array (e.g., `let allEvents = []`). Write a function `normalizeData(rawData)` that loops through the API response and returns a cleaned array of objects. Convert the raw `StartDateTime` string into a standard JavaScript `Date` object for easier sorting later. Provide fallbacks for missing `LocationName` or `Title`.
* **Verification:** Log the normalized array. Verify that the date objects are valid and missing locations say "TBD" instead of `undefined` or `null`.

### Phase 5: Dynamic DOM Rendering
* **Action:** Write a function `renderEvents(eventsArray)`. It should clear the `<main>` container, loop through the array, and dynamically create HTML strings or DOM nodes for each event card, injecting the cleaned data. Remove the hardcoded Phase 2 HTML.
* **Verification:** The webpage should now populate with real BYU event data. Scroll through the page to ensure all rendered fields (Title, Date/Time, Location, Price) format correctly.

### Phase 6: Search & Filter Logic
* **Action:** Add a search `<input>` and a "Free Only" checkbox to the HTML. In `app.js`, add event listeners (`'input'` or `'change'`) to these elements. Create a `filterEvents()` function that reads the state of these inputs, filters the `allEvents` array, and passes the result to `renderEvents()`.
* **Verification:** Type "basketball" into the search bar; the UI should instantly update to show only matching events. Check the "Free Only" box; paid events should disappear.

### Phase 7: Sorting Logic
* **Action:** Add a `<select>` dropdown in HTML with options for "Date (Earliest First)", "Date (Latest First)", and "Location (A-Z)". Add an event listener to trigger a `sortEvents(filteredArray)` function before rendering. Use JavaScript's `Array.prototype.sort()` based on the selected criteria.
* **Verification:** Change the dropdown options. Ensure the UI updates and the events are genuinely sorted chronologically or alphabetically based on your selection.

### Phase 8: Loading, Empty States, & Error Handling
* **Action:** 1. Add a CSS loading spinner or "Loading events..." text that displays before the fetch completes.
    2. Add an "Empty State" UI if `renderEvents()` receives an empty array (e.g., "No events match your filters").
    3. Wrap the fetch in a `try/catch` block. If it fails, display a user-friendly error message on the screen.
* **Verification:** 1. Throttle your network speed in the browser DevTools to visually verify the loading state. 
    2. Type gibberish into the search bar to verify the empty state appears. 
    3. Temporarily break the API URL in the code to verify the error message appears on the screen instead of silently failing in the console.

### Phase 9: Accessibility (a11y) & Polish
* **Action:** Audit the HTML. Ensure buttons/inputs have `aria-labels` if they lack text. Ensure color contrast ratios meet WCAG AA standards (especially gray text on dark backgrounds). Ensure the entire app can be navigated using only the `Tab` and `Enter` keys.
* **Verification:** Unplug your mouse and try to use the search, filters, and sorting dropdown using only your keyboard. Run a Lighthouse Accessibility audit in Chrome DevTools to check for missing semantic tags or contrast issues.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrewblodgett)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andrewblodgett)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
