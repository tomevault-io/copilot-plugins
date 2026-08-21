## android-currency-calculator

> Android travel calculator app with live currency conversion, tip, fuel, and distance tools. Built with Kotlin + Jetpack Compose. Published on Play Store as TripCalc (`com.tripcalc.app`).

# CalcApp (TripCalc)

Android travel calculator app with live currency conversion, tip, fuel, and distance tools. Built with Kotlin + Jetpack Compose. Published on Play Store as TripCalc (`com.tripcalc.app`).

## Project Structure

```
app/src/main/java/com/tripcalc/app/
├── MainActivity.kt                  # Entry point — just calls AppScreen(), no theme wrapper
├── ui/
│   ├── AppTheme.kt                  # DarkModePref, AccentScheme, AppColors, buildAppColors, LocalAppColors
│   ├── CalculatorScreen.kt          # AppScreen (root), CalculatorScreen, CurrencySelector, ButtonGrid, dialogs, About
│   ├── CalculatorViewModel.kt       # CalculatorUiState, CalculatorAction, ConversionMode, DistancePair, state machine
│   ├── HistoryScreen.kt             # Calculation history list, tap-to-restore
│   ├── SettingsScreen.kt            # Appearance, haptic, custom rates, card fee markup, tab visibility
│   └── theme/Theme.kt               # CalcAppTheme — Material3 colour scheme (called from AppScreen, not MainActivity)
└── data/
    ├── ExchangeRateApi.kt           # Retrofit interface + response model (open.er-api.com)
    └── ExchangeRateRepository.kt    # Fetch/cache rates + persist all prefs via DataStore
docs/
├── privacy-policy.html              # Hosted via GitHub Pages for Play Store
├── store-description-short.txt      # 65-char Play Store tagline
└── store-description-long.txt       # Full Play Store description
```

## Build & Install

```bash
# Build debug APK
cd ~/calcApp && ./gradlew assembleDebug

# Connect and install over WiFi ADB
# Port changes every session — user provides it and it is saved to ~/.adb_port
PORT=$(cat ~/.adb_port) && ~/android/platform-tools/adb connect 192.168.68.128:$PORT
~/android/platform-tools/adb install -r app/build/outputs/apk/debug/app-debug.apk
```

APK output: `app/build/outputs/apk/debug/app-debug.apk`

## Key Decisions

- **Exchange rate API**: `open.er-api.com/v6/latest/USD` — free, no API key, 161 currencies
- **Cache TTL**: 24 hours via DataStore; `forceRefresh = true` bypasses it
- **Base currency**: USD — all rates stored relative to USD, cross-rates computed as `value * homeRate / localRate`
- **Input direction**: entered value is treated as **local currency**; home currency equivalent shown alongside
- **Card fee markup**: `cardMarkupPercent: Double` (0–5%, default 0) in `CalculatorUiState`; applied as `markupFactor = 1.0 + percent / 100.0` on `toValue` in `updateCurrencyDisplay()`; shown in rate label when > 0; persisted via DataStore `"card_markup_percent"`
- **Custom rates**: `CustomRateEntry(base, rate)` = "1 base = rate target"; `effectiveUsdRate()` converts to USD-relative for math
- **History**: logged on every `=` press into `List<HistoryEntry>` (max 50), persisted as JSON in DataStore
- **Offline indicator**: `isOffline: Boolean` in `CalculatorUiState`; set true when network request fails
- **Theming**: `LocalAppColors` CompositionLocal provides `AppColors` built from `(effectiveIsDark, AccentScheme)`; `CalcAppTheme` is applied inside `AppScreen` (not `MainActivity`) so the VM's `darkModePref` controls it
- **Currency picker**: uses `Dialog` + `LazyColumn` — **never** `DropdownMenu` + `LazyColumn` (nested vertical scroll crash)
- **`currencyName(code)`**: cached `HashMap` helper in `CalculatorScreen.kt`; use it instead of `JavaCurrency.getInstance().displayName` inline
- **Currency persistence**: all prefs (currencies, recents, custom rates, history, haptic, dark mode, accent, card markup, tip %, fuel units, distance pairs, enabled modes) saved to DataStore
- **Version string**: `BuildConfig.VERSION_NAME` in `CalculatorScreen.kt` (About page) — auto-synced from `versionName` in `app/build.gradle.kts`; requires `buildConfig = true` in `buildFeatures` (AGP 8+)
- **Layout scaling**: `CalculatorScreen` Column uses `fillMaxSize()` (no `verticalScroll`); `ButtonGrid` takes `Modifier.weight(1f)` in its parent; ButtonGrid rows use `Modifier.weight(1f).fillMaxWidth()`; `CalcButton` uses `fillMaxHeight()` — this makes buttons scale proportionally on all screen sizes
- **Tab labels**: `maxLines = 1, overflow = TextOverflow.Ellipsis` on SegmentedButton text prevents two-line wrapping on small/large screens

## Colour System

All colours come from `LocalAppColors.current` — never hardcode hex values in composables.

```kotlin
val colors = LocalAppColors.current
// Key fields: background, surface, buttonDigit, buttonDigitContent,
// buttonFunction, buttonFunctionContent, textPrimary, textSecondary,
// textMuted, divider, inputBorder, operator, operatorContent,
// equals, equalsContent, fromAmountColor, toAmountColor,
// errorColor, positiveColor, negativeColor, warningColor, promoColor
```

## ADB Setup Notes

- Android SDK at `~/android`, platform-tools at `~/android/platform-tools`
- WiFi ADB preferred; Samsung device at `192.168.68.128`
- Connect port changes every session — shown on device's Wireless Debugging screen
- If `adb connect` says "Connection refused" but the device was connected earlier in the session, `adb install` may still succeed
- Pairing codes expire quickly — have ports ready before running pair command
- If debug APK fails with `INSTALL_FAILED_UPDATE_INCOMPATIBLE` (signature mismatch vs Play Store version), uninstall first: `adb uninstall com.tripcalc.app`

---
> Source: [richw100/android-currency-calculator](https://github.com/richw100/android-currency-calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
