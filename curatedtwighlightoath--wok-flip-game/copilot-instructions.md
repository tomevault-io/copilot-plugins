## wok-flip-game

> A mobile casual game built in React Native focused on timing-based wok flipping mechanics. The game is designed to monetize through **in-app purchases (IAP) and ads**.

# Wok Flip Game - Project Context for Claude

## Project Overview

A mobile casual game built in React Native focused on timing-based wok flipping mechanics. The game is designed to monetize through **in-app purchases (IAP) and ads**.

## Core Gameplay Loop

- Ingredients fall from the top of the screen
- The player slides the wok left/right to position it and taps to flip
- A 200ms timing window on each flip catches ingredients inside the wok's bowl
- Each level has an assigned recipe to complete
- **Burn mechanic**: the player must flip at least once every 5 seconds or food burns
- **Wrong ingredient avoidance**: non-recipe ingredients also fall, requiring strategic flip timing to avoid catching them
- Catching consecutive correct ingredients builds a combo multiplier

## Gameplay Features Discussed

- **Combo system**: consecutive correct catches multiply score/coin drops
- **Sizzle meter**: burn threshold creeps closer the longer you wait, creating dynamic difficulty
- **Oil splatter events**: random hazards that temporarily shrink the flip timing window
- **Recipe mastery progression**: repeat dishes to unlock upgraded versions (e.g., fried rice → spicy fried rice → chef's special)
- **Seasonal/regional cuisine themes**: event content refreshes (e.g., Lunar New Year packs)

## Monetization Design

| Mechanic | Monetization Hook |
|---|---|
| Burn timer | "Fireproof wok" consumable IAP |
| Wrong ingredient mistakes | Lives system; extra lives as IAP |
| Failed level | Revive via rewarded video ad |
| Between levels | Interstitial ad placement |
| Premium cuisines | Recipe book with locked cuisine packs (cosmetic IAP) |

## Tech Stack

**Framework**: React Native + Expo (bare workflow with native modules)

**Key Libraries**:
- `@shopify/react-native-skia` v2.4 — Skia canvas rendering for all game visuals
- `react-native-reanimated` v4 — UI-thread animations via worklets and `useFrameCallback`
- `react-native-gesture-handler` v2 — tap (flip) and pan (slide) gesture detection
- `react-native-google-mobile-ads` v16 — ad integration (test app ID configured)
- `react-native-iap` v14 — in-app purchase integration

## Architecture

### Performance Model
All gameplay runs on the **UI thread** — zero JS bridge crossings during active play:
- **Game loop**: `useFrameCallback` from Reanimated (runs as a worklet)
- **Ingredient pool**: fixed pool of 12 shared value slots (no React re-renders during gameplay)
- **Gestures**: tap and pan run as worklets, directly mutate shared values
- **Rendering**: Skia `<Canvas>` reads shared values via `useDerivedValue`

### Key Constants
- `MAX_INGREDIENTS = 12` — ingredient pool size
- `GRAVITY = 420` px/s² — fall speed
- `SPAWN_INTERVAL_MS = 1200` — new ingredient every 1.2s
- `CATCH_WINDOW_FRAMES = 12` — ~200ms catch window after flip
- `WOK_RATIO = 0.3` — wok is 30% of screen width (handle to handle)

### File Structure
```
src/
  screens/
    GameScreen.tsx        — gesture handling (tap + pan), shared value ownership
  components/
    GameCanvas.tsx        — Skia canvas, game loop (spawn/physics/catch detection)
    Wok.tsx               — wok drawing with dynamic sizing, flip animation
    Ingredient.tsx        — ingredient rendering from shared value slots
  data/
    ingredients.ts        — 8 ingredient definitions (color, radius)
```

### Data Flow
```
GameScreen (gestures)
  ├── flipProgress, flipTranslateY → Wok (animation)
  ├── flipActive → GameCanvas (catch window trigger)
  └── wokX → Wok (position) + GameCanvas (catch zone)

GameCanvas (useFrameCallback worklet)
  ├── spawns ingredients into shared value pool
  ├── updates physics (gravity) per frame
  └── checks catch zone overlap during 200ms window
```

## Build & Run

```bash
# Set environment (cmd)
set JAVA_HOME=C:\Program Files\Android\Android Studio\jbr
set ANDROID_HOME=C:\Users\Kspider\AppData\Local\Android\Sdk

# Build and run
npx expo run:android
```

- `android/gradle.properties` has `org.gradle.java.home` set to Android Studio's bundled JDK 21
- `android/local.properties` has `sdk.dir` pointing to the Android SDK
- First build takes ~6 min (compiles native C++ for Skia); subsequent builds are fast
- Hot reload works for JS changes; native changes require rebuild

## Project Status

**Core prototype working.** Implemented so far:
- Wok rendering with satisfying spring flip animation
- Ingredient falling with gravity physics
- Catch detection with 200ms timing window
- Wok sliding via pan gesture (clamped to screen bounds)
- All gameplay on UI thread (smooth 60fps)

**Next up:**
- Further optimization and polish
- Score/UI overlay
- Burn timer mechanic
- Recipe system (correct vs wrong ingredients)
- Visual effects for catches
- Image assets for ingredients and wok (replacing Skia primitives)

---
> Source: [Curatedtwighlightoath/wok-flip-game](https://github.com/Curatedtwighlightoath/wok-flip-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
