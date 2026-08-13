## gymtracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start          # Start Expo dev server (scan QR with Expo Go)
npm run android    # Start on Android emulator/device
npm run ios        # Start on iOS simulator/device
npm run web        # Start in browser
```

No test suite or linter is configured.

## Environment Setup

A `.env` file is required at the project root:

```
ANTHROPIC_API_KEY=your_key_here
```

This is loaded via `react-native-dotenv` and imported as `import { ANTHROPIC_API_KEY } from '@env'`.

## Architecture

React Native / Expo app with three tab screens. All state is local React hooks; persistence is `AsyncStorage` only — there is no backend or database. Code is organized by feature under `src/features/`, with shared components/hooks/utils under `src/shared/`.

### Navigation

`App.js` → `src/navigation/AppNavigator.js` → React Navigation bottom tab with three screens:
- **Macros** → `src/features/macroTracker/MacroTrackerScreen.js`
- **Reps** → `src/features/repCounter/RepCounterScreen.js`
- **Calculator** → `src/features/calculator/CalculatorScreen.js`

`AppNavigator` wraps everything in `ThemeProvider` (`src/shared/context/ThemeContext.js`), which supplies the app's single light color palette via `useTheme()`.

### Macro Tracker

`useMacroTracker` (`src/features/macroTracker/hooks/useMacroTracker.js`) owns all macro state; the screen passes handlers down as props. Key state objects:

| State | AsyncStorage key | Description |
|---|---|---|
| `customFoods` | `CUSTOM_FOODS` | User-defined foods with known macros |
| `dailyLog` | `DAILY_LOG` | `{ [dateStr]: { items: { [id]: { item, count } }, totals } }` |
| `historyByDate` | `HISTORY_BY_DATE` | `{ [dateStr]: [{ foodId, key, items }] }` — GPT/custom food entries per day |
| `gptCache` | `GPT_CACHE` | `{ [searchKey]: { searchKey, foodId, items } }` — cached GPT responses |
| `goals` | `GOALS` | `{ calories, protein, carbs, fats }` targets |

Food lookup flow: user types → check `gptCache` → if miss, call the Claude API (`claude-haiku-4-5`, structured outputs, via raw `fetch` — the `@anthropic-ai/sdk` package is deliberately NOT used because it imports `node:fs`, which Metro cannot bundle for native) from `services/gptService.js` → normalize via `utils/gptUtils.js` → store in cache and add to `historyByDate`. (File/state names keep the legacy "gpt" prefix.) There is also a scan-label flow: photo → `utils/imageUtils.js` (resize/compress via expo-image-manipulator) → `fetchNutritionFromImage`.

`dailyLog` and `historyByDate` serve different purposes: `dailyLog` tracks item counts and running totals for display; `historyByDate` preserves the original GPT entries (used by `DailyControls` to render each meal entry with +/- controls).

### Rep Counter

`useRepCounter` (`src/features/repCounter/hooks/useRepCounter.js`) owns all data in a single `data` state object stored at `REP_COUNTER_DATA`:

```
data: {
  [groupName]: {
    [exerciseName]: [
      { date: "YYYY-MM-DD", sets: [{ reps, weight }], notes: "" }
    ]
  }
}
```

Screen renders as a drill-down: Categories → Exercises → Set log. Navigated by `selectedGroup` / `selectedExercise` state (null = list view). Day notes are stored separately at `DAY_NOTES`.

### Module Aliases

`jsconfig.json` sets `baseUrl: "src"`, so all imports resolve from `src/`. Examples:

```js
import { useTheme } from "shared/hooks/useTheme";
import { Button } from "shared/components/Button";
import MacroTrackerScreen from "features/macroTracker/MacroTrackerScreen";
```

### Date Formats

- `DD/MM/YY` — display format and primary key for macro tracker state (`todayString()`, `selectedDate`)
- `YYYY-MM-DD` — ISO format used internally in rep counter and for calendar library; convert with `dmyToIso` / `isoToDmy` from `src/shared/utils/dateUtils.js`

### Styling

The color palette lives in `src/shared/constants/colors.js` (`COLORS`, also exported as `themes.light`). Layout tokens are in `src/shared/constants/styles.js` (`SPACING`, `FONT_SIZE`, `FONT_WEIGHT`, `BORDER_RADIUS`, `SHADOW`, `CONTROL_HEIGHT`). Components get colors via `useTheme()` and per-feature `createThemedStyles(colors)` factories (e.g. `src/features/macroTracker/macroTrackerStyles.js`) — never import `COLORS` directly in new UI.

---
> Source: [NaDeans/GymTracker](https://github.com/NaDeans/GymTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
