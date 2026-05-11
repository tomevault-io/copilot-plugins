## color-challenge

> Web version of the Color Snap daily color challenge game. Built as a single-page React app with client-side image analysis (canvas API) and localStorage persistence. No backend required.

# CLAUDE.md — Color Snap (Web)

## Overview

Web version of the Color Snap daily color challenge game. Built as a single-page React app with client-side image analysis (canvas API) and localStorage persistence. No backend required.

**Stack:** React 19, Vite, vanilla CSS (inline JSX styles)

## Build & Run

```bash
cd color-challenge-app
npm install
npm run dev        # Vite dev server (http://localhost:5173)
npm run build      # Production build → dist/
npm run preview    # Preview production build
npm run lint       # ESLint
npm test           # Run unit tests (vitest)
npm run test:watch # Watch mode
```

## Architecture

Two source files:

- **`src/logic.js`** — All pure functions (palette, seeded RNG, color generation, hex/RGB/HSL conversion, streak calculation, import/export parsing). Extracted for testability — no DOM or React dependencies.
- **`src/App.jsx`** — React UI, screens, Canvas-based image analysis, localStorage persistence, browser-specific import/export wrappers. Imports everything from `logic.js`.

### Screens
- **ChallengeScreen** — Daily color + photo upload + analysis
- **ResultsScreen** — Pass/fail results with share functionality
- **CalendarScreen** — Monthly calendar with streak tracking + data export/import

### Key Functions (in `logic.js`)
- `seededRandom(seed)` — Deterministic RNG (32-bit hash)
- `getColorForDate(dateStr)` — Returns today's assigned color from the palette
- `hexToRgb(hex)` / `rgbToHsl(r, g, b)` — Color space conversion
- `rgbDistance(hex1, hex2)` — Euclidean RGB distance for consecutive-day diversity
- `calculateStreak(submissions)` — Walks backward from today counting consecutive completions
- `parseImportJson(jsonString, existingSubmissions)` — Validates and merges imported data
- `buildExportObject(submissions)` — Builds universal export JSON object

### Key Functions (in `App.jsx`, browser-only)
- `analyzeImage(file, targetHex, options)` — Canvas-based HSL pixel matching
- `exportData(submissions)` — Triggers browser download of export JSON
- `importData(file, existingSubmissions)` — FileReader wrapper around `parseImportJson`

### Data Persistence
```javascript
localStorage key: "color-challenge-data"
// Structure: { submissions: { "YYYY-MM-DD": { completed, difficulty, results, passCount } } }
```

## Critical: Cross-Platform Color Algorithm

The daily color assignment **must stay in sync** with the Android app at `../color-challenge-android/`. Both use:

- **Seed format:** `"YYYY-MM-colorchallenge-monthly-v1"`
- **RNG:** `h = Math.imul(31, h) + charCode | 0`, then MurmurHash-like mixing with `0x45d9f3b`
- **Deduplication:** No repeated colors within same calendar month
- **Palette:** Must match Android's 114 colors exactly (in `ColorPalette.kt`)

If you change the algorithm here, you **must** also update `../color-challenge-android/app/src/main/java/com/colorsnap/data/DailyColorGenerator.kt`.

## Critical: Cross-Platform Color Analysis

The photo analysis algorithm **must stay in sync** with the Android app. Both use identical HSL-based pixel matching with the same tolerances:

- **Easy:** 3 photos, ±25° hue, ±0.55 saturation, ±0.45 lightness, ≥2% pixel match threshold
- **Hard:** 5 photos, ±15° hue, ±0.35 saturation, ±0.30 lightness, ≥4% pixel match threshold
- **MIN_SATURATION:** 0.15 — pixels below this are filtered as noise
- **Downsampling:** Max 400px dimension
- **Denominator:** comparable pixels (those above MIN_SATURATION), falling back to total pixels

If you change the `DIFFICULTY` constant in `logic.js`, you **must** also update `../color-challenge-android/app/src/main/java/com/colorsnap/data/ChallengeRepository.kt` (the `DIFFICULTIES` map).

## Data Export/Import

Universal JSON format shared with Android (and future iOS). Photos are excluded (device-specific URIs are not portable).

- **Export:** `buildExportObject()` in `logic.js` builds the JSON; `exportData()` in `App.jsx` triggers browser download
- **Import:** `parseImportJson()` in `logic.js` validates, filters invalid date keys, and merges; `importData()` in `App.jsx` wraps it with FileReader
- **Merge strategy:** Higher `passCount` wins; ties favor local data
- **Format:** `{ colorSnap: { version: 1, exportedFrom: "web", submissions: {...}, shields: {...} } }`

## Unit Tests

Vitest tests covering all pure logic in `logic.js`. Run with `npm test`.

```
src/__tests__/
├── colorConversion.test.js  # 13 tests — hexToRgb, rgbToHsl, rgbDistance
├── seededRandom.test.js     #  6 tests — determinism, range, seed formats
├── colorGeneration.test.js  # 12 tests — getColorForDate, palette integrity, deduplication, consecutive distance
├── streak.test.js           # 13 tests — calculateStreak (empty, consecutive, gaps, cross-month)
├── dateUtils.test.js        #  9 tests — getLocalDateStr, getTimeRemaining
├── dataTransfer.test.js     # 23 tests — parseImportJson, buildExportObject, merge logic, round-trip
└── difficulty.test.js       #  6 tests — DIFFICULTY constants match Android source of truth
```

**90 tests total.** All are pure Node.js tests with no browser or DOM required. Uses `vi.useFakeTimers()` to pin dates for streak/time tests.

### Key test areas

- **Color conversion:** RGB/HSL round-trip accuracy, known color values, distance symmetry
- **Color generation:** Determinism, no duplicates within a month, consecutive-day RGB distance ≥ 100, leap year handling
- **Streak calculation:** Empty cases, consecutive days, gap breaks, cross-month boundaries, non-completed entries
- **Data transfer:** Version validation, date key filtering, merge strategy (higher passCount wins, ties favor local), overlap detection, round-trip export→import, error cases
- **Difficulty constants:** Exact match with Android tolerances (Android is source of truth)

## Deployment

Static site — deploy `dist/` to any CDN (Vercel, Netlify, GitHub Pages, etc.).

---
> Source: [AdamLukas17/color-challenge](https://github.com/AdamLukas17/color-challenge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
