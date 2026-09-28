## indraprastha-meal-planner

> A **pure HTML/CSS/JS single-file offline Indian meal planner** for a family home. No frameworks, no build tools — everything is in `index.html`. Data is stored in `localStorage` under key `indraprastha-v2`.

# Indraprastha — Weekly Meal Planner

## What this app is
A **pure HTML/CSS/JS single-file offline Indian meal planner** for a family home. No frameworks, no build tools — everything is in `index.html`. Data is stored in `localStorage` under key `indraprastha-v2`.

## How to run
- Server: `python -m http.server 8000 --directory "D:\Github Projects\"` (or use `.claude/serve.bat`)
- Laptop: `http://localhost:8000/Indraprastha-Meal-Planner/index.html`
- Phone (same Wi-Fi): `http://192.168.0.170:8000/Indraprastha-Meal-Planner/index.html`
- GitHub: `https://github.com/AditiSamarjeet/Indraprastha-Meal-Planner` (branch: main)
- GitHub Pages: `https://aditisamarjeet.github.io/Indraprastha-Meal-Planner/` (needs repo to be public)

## Three modes
| Mode | Access | Language |
|------|--------|----------|
| Family | Default | English |
| Helper | Bottom bar | 100% Marathi |
| Admin | PIN: **1234** | English |

## Key features built
- **Weekly Planner** — 4 weeks, all meals (breakfast/lunch/snacks/dinner)
- **Save-while-typing** — type a new dish in planner → ➕ chip saves it to options permanently
- **📦 Preset dish packs** — 600+ Indian dishes, open via Customise view
- **📋 Bulk add** — paste comma-separated dish names
- **Today's Menu** — family/helper/admin views with daily staff status bar
- **Grocery List** — 318+ items with qty/brand, category management
- **Buying List** — checked items ready for shopping
- **Meal Calendar (rcalendar)** — set special menus, festivals, staff availability per date
- **Helper Schedule (vेळापत्रक)** — monthly calendar showing all staff absent/half-day days
- **Multi-staff availability** — Admin marks each staff member per date (present/absent/half-day + note); appears on Today's Menu (all modes) and Weekly Planner day headers
- **Full Marathi helper mode** — phonetic Devanagari fallback for unknown English words via `_phoneticDev()`
- **Customise Options** — manage dish options per category with Presets + Bulk add
- **Daily Essentials** — recurring daily items
- **History, Recipes, Backup** — admin-only views

## Key JS functions
- `applyMode(mode)` — switches between 'family'/'helper'/'admin'
- `showView(name)` — navigates to a view
- `mr(s)` — translates English → Marathi (MR_MAP dict + word-by-word + phonetic fallback)
- `_phoneticDev(word)` — Roman → Devanagari phonetic transliteration
- `ddFilter(inp)` — filters planner dropdown; shows ➕ Save chip for new dishes
- `ddSaveToOpts(chip,e)` — saves new dish to opts array
- `buildStaffBar(dateKey, compact)` — renders staff status chips for a date
- `renderHelperSchedule()` — renders the वेळापत्रक calendar with all staff
- `hscalSetStatus(dateKey, memberId, status)` — set staff availability
- `saveToStorage(toast)` — persists to localStorage
- `mkSel(k, val, day, field)` — builds planner dropdown cell (uses `data-optkey`)

## Data model (localStorage: indraprastha-v2)
```js
{
  opts,          // dish options per category
  essentials,    // daily essentials categories
  weeks,         // 4 weeks × 7 days × all meal fields
  groceryList,   // categories with items
  rcalData,      // calendar special menus/festivals per date
  dailyTasks,    // tasks/to-buy per date key
  mealHistory,
  recipes,
  helperSchedule, // legacy single-helper schedule (migrated to staffSchedule)
  staffSchedule: {
    members: [{id, name, role, color, emoji}],
    dates: { dateKey: { memberId: {status, note} } }
  }
}
```

## Important CSS classes
- `.admin-only` / `.admin-only-block` — hidden unless `body.admin-mode`
- `.en` / `.mr` — language toggle via `body.marathi-mode`
- `.meal-dd` — planner dropdown container (has `data-optkey`, `data-day`, `data-field`)
- `.meal-dd-save` — the ➕ Save chip in dropdowns
- `.staff-bar` / `.staff-chip` — staff status bar in Today's Menu
- `.hscal-cell` / `.hscal-sc` — helper schedule calendar cells/chips

## Views available per mode
- **Admin**: todaymenu, planner, rcalendar, grocery, buyinglist, mealderived, history, customise, essentials, recipes, backup, hlprschedule
- **Family**: todaymenu, planner, rcalendar, grocery, buyinglist, mealderived, essentials
- **Helper**: todaymenu, hlprschedule, grocery, buyinglist, essentials

## Dinner cuisine system
`DINNER_CUISINES` object has 8 cuisines (indian, italian, continental, asian, ricebased, mexican, chinese, custom) each with `labelMr` and `fields[{key, label, labelMr}]`.

## Known constraints
- Single HTML file — no build step, no imports
- localStorage only — no backend, no sync between devices
- Export/Import JSON to sync data between devices
- Service workers don't work on local network HTTP (only localhost/HTTPS)
- GitHub Pages needs repo to be **public** for free tier

## Pending / nice to have
- PWA manifest + service worker for "Add to Home Screen" and offline support
- Data sync between devices (currently manual Export/Import)
- Monthly attendance register / summary view
- Push notifications for daily meal reminders

---
> Source: [AditiSamarjeet/Indraprastha-Meal-Planner](https://github.com/AditiSamarjeet/Indraprastha-Meal-Planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
