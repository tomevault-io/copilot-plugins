## shop-sales-reporting

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**HaFiz PhotoCopy — Sales Dashboard** is a zero-dependency single-page application for tracking daily sales and expenses of a photocopy shop. All code lives in one HTML file (`hafiz_photocopy_dashboard.html`) with embedded CSS and JavaScript. Currency is in Pakistani Rupees (Rs).

## Architecture

- **Single-file SPA** — no build step, no package manager, no framework, no bundler. Open the file in a browser or serve with any static file server.
- **State management** — a plain `state` object holds `months[]`, `dataByMonth{}`, `selectedScope`, and `loaded` flag. No reactive framework.
- **Storage** — uses `window.storage.get/set(key, shared)` (a platform-provided API, likely from Glitch or similar shared-hosting service) for cross-user persistence. Falls back to hardcoded `INITIAL_DATA` in offline mode.
- **Rendering** — all DOM manipulation is imperative (innerHTML assignments, event listeners). No virtual DOM.
- **Charts** — custom SVG rendering with zero charting libraries. Three chart types: bar+line trend chart, donut expense chart, and day-of-week bar chart.

### Key code sections (all in one file)

| Section | Lines | Description |
|---|---|---|
| CSS | 10–203 | Theming with CSS custom properties, responsive grid, skeleton loading, header scanning-light animation |
| HTML | 208–325 | Header, KPI cards, chart containers, table, modal form, toast notification |
| State & helpers | 333–359 | `state` object, `fmtMoney()`, `monthLabel()`, `weekdayFromDate()`, `showToast()` |
| Storage | 362–441 | `initStorage()`, `upsertDay()`, `deleteDay()`, `persistMonth()`, `persistMonthsList()` |
| Derived data | 444–457 | `scopedDays()`, `monthTotals()` |
| Rendering | 460–760 | `renderKPIs()`, `renderTrendChart()`, `renderExpenseChart()`, `renderDowChart()`, `renderTable()`, `renderAll()` |
| Modal/Form | 762–848 | Add/edit entry form, delete confirmation, form validation |
| Boot | 856–860 | IIFE that initializes storage and renders the dashboard |

## Data Model

Each day entry:

```js
{
  date: "2026-05-01",       // ISO date string
  day: "Friday",            // weekday name, computed from date
  sale: 300,                // total sales amount
  note: null,               // optional string (e.g., "Eid", "NO LIGHT")
  bestSeller: null,          // optional string describing top-selling item
  shopExp: 0,               // shop expenses (rent, toner, bills, etc.)
  shopExpDetail: null,      // optional string describing shop expense
  otherExp: 0,              // personal/other expenses
  otherExpDetail: null      // optional string describing other expense
}
```

Days are grouped by month key (`"2026-05"`) in `state.dataByMonth`. `state.months` holds sorted month keys.

Net profit = `sale - shopExp - otherExp`.

## Development

### How to run locally

```bash
# Serve with Python (no install needed on most systems)
python3 -m http.server 8000
# Or with Node
npx serve .
# Or just open the HTML file directly in a browser
```

### Common tasks

- **Add or edit a day entry** — click "+ Add day" button, fill the modal form. The form auto-suggests the next date after the last recorded day.
- **View entry details** — click a table row to expand/collapse details (best seller, expense descriptions).
- **Edit an entry** — expand the table row, click "Edit entry".
- **Delete an entry** — expand the table row, click "Delete entry".
- **Month selector** — switches the dashboard scope between "All months" and a specific month.

### Testing

There are no automated tests. Verify changes by serving the file and manually testing CRUD operations (add, edit, delete entries) and month filtering.

## Key Patterns & Conventions

- **Monetary formatting** — use `fmtMoney(n)` which produces `"Rs 1,234"` format.
- **Date handling** — all date strings are ISO (`"YYYY-MM-DD"`). Parse with `new Date(dateStr + "T00:00:00")` to avoid timezone issues.
- **Weekday order** — `WEEKDAY_ORDER` starts with Monday: `["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"]`.
- **Month labels** — `monthLabel("2026-05")` produces `"May 2026"`.
- **XSS prevention** — use `escapeHtml()` for any user-supplied values rendered to HTML.
- **Toast notifications** — use `showToast(msg)` for ephemeral feedback (2.2s auto-hide).
- **State changes** — after any mutation (add/edit/delete), call `renderAll()` to refresh the full dashboard, then `showToast()` for confirmation.
- **Scope awareness** — `renderKPIs()` shows a delta indicator (% change vs prior month) when a specific month is selected.
- **Changes to the page** — modify the HTML within the `<div id="app">` container. The style section is at the top. The chart containers, KPI grid, and table panel are `div` elements with `id` attributes referenced in JavaScript.

---
> Source: [asad-bukhari/shop-sales-reporting](https://github.com/asad-bukhari/shop-sales-reporting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
