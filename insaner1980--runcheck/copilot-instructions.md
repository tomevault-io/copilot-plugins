## runcheck

> runcheck is a native Android app (Kotlin + Jetpack Compose) that monitors device health across four categories: battery, network, thermal, and storage. It provides real-time diagnostics, a unified health score, long-term trend tracking, and storage cleanup tools.

# CLAUDE.md — runcheck Project Instructions

## Project Overview

runcheck is a native Android app (Kotlin + Jetpack Compose) that monitors device health across four categories: battery, network, thermal, and storage. It provides real-time diagnostics, a unified health score, long-term trend tracking, and storage cleanup tools.

## Tech Stack

- **Language:** Kotlin 2.3.0 (via AGP 9.1.0 built-in Kotlin)
- **UI:** Jetpack Compose with Material 3 (BOM 2026.03.00), single dark theme
- **Min SDK:** 26 (Android 8.0)
- **Target SDK:** CinnamonBun beta (Android 17)
- **Compile SDK:** CinnamonBun beta (Android 17)
- **Architecture:** MVVM with Clean Architecture layers (data → domain → ui)
- **Database:** Room 2.8.4 for local historical data
- **Async:** Kotlin Coroutines 1.10.2 + Flow
- **DI:** Hilt 2.59.2
- **Charts:** Custom Compose components (TrendChart with axes/grid/tooltip/quality zones, AreaChart, HeatStrip, SegmentedBar)
- **Build:** Gradle 9.4.0 with Kotlin DSL, AGP 9.1.0, KSP 2.3.1
- **Localization:** English only (Finnish translations preserved in git history for future use)

## Project Structure

```
app/src/main/java/com/runcheck/
├── data/
│   ├── appusage/      # App battery usage data source
│   ├── battery/       # BatteryManager wrappers, BatteryDataSourceFactory,
│   │                  #   GenericBatterySource + manufacturer-specific sources
│   │                  #   (Android14, Samsung, OnePlus), BatteryCapacityReader
│   ├── billing/       # BillingManager (lifecycle-aware billing service),
│   │                  #   ProStatusCache
│   ├── charger/       # Charger comparison data
│   ├── device/        # Device detection, DeviceProfile, DeviceProfileProvider,
│   │                  #   DeviceCapabilityManager
│   ├── db/            # Room database (RuncheckDatabase with inner Converters, RoomTransactionRunner)
│   │   ├── dao/       # Room DAOs (Battery, Network, Thermal, Storage, Charger, etc.)
│   │   └── entity/    # Room entities (BatteryReadingEntity, etc.)
│   ├── export/        # Data export functionality
│   ├── insights/      # Persisted insights repository, debug seed helpers
│   ├── network/       # ConnectivityManager, TelephonyManager, SpeedTestService,
│   │                  #   LatencyMeasurer, NetworkRepositoryImpl, SpeedTestRepositoryImpl
│   ├── preferences/   # DataStore preferences
│   ├── storage/       # MediaStoreScanner, StorageCleanupHelper,
│   │                  #   CleanupPagingSource, StorageCleanupRepositoryImpl
│   └── thermal/       # ThermalManager, thermal/throttling repositories
├── domain/
│   ├── insights/      # Insight models (InsightModels.kt), rules, engine (+ ranking policy)
│   ├── model/         # Domain models (BatteryState, NetworkState, StorageState, etc.)
│   ├── usecase/       # Business logic (37 use cases)
│   ├── repository/    # Repository interfaces
│   └── scoring/       # Health score algorithm
├── ui/
│   ├── home/          # Single home screen (hub) + ViewModel
│   ├── insights/      # Dedicated Insights screen + ViewModel
│   ├── battery/       # Battery detail screen + ViewModel + session stats
│   │                  #   + BatteryInfoContent, BatteryInfoCards
│   ├── network/       # Network detail + SpeedTest screens + ViewModel
│   │                  #   + NetworkInfoContent, SpeedTestInfoContent, NetworkInfoCards
│   ├── thermal/       # Thermal detail screen + ViewModel + session min/max
│   │                  #   + ThermalInfoContent, ThermalInfoCards
│   ├── storage/       # Storage detail screen + ViewModel + cleanup/
│   │                  #   (CleanupScreen, CleanupViewModel, FileListItem, CategoryGroup)
│   │                  #   + StorageInfoContent, StorageInfoCards
│   ├── charger/       # Charger comparison screen + ViewModel
│   ├── appusage/      # App battery usage screen + ViewModel
│   ├── learn/         # Learn section — article catalog, list screen, detail screen
│   ├── settings/      # Settings screen + ViewModel
│   ├── pro/           # Pro upgrade screen, trial UI, purchase flow
│   ├── theme/         # Dark theme, color tokens, typography, spacing, motion tokens
│   ├── common/        # UiText, UiFormatters (formatPercent, formatTemp, etc.)
│   ├── chart/         # ChartHelpers (quality zones, axis labels, qualityZoneColorForValue),
│   │                  #   ChartModels, ChartRenderModel, ChartAccessibility
│   ├── fullscreen/    # FullscreenChartScreen + ViewModel (landscape-only)
│   ├── components/    # 34 shared composables (see Components below)
│   │   └── info/      # InfoSheetContent, InfoIcon, InfoBottomSheet, InfoCard, InfoCardCatalog, CrossLinkButton
│   └── navigation/    # NavGraph + Screen sealed class (push-based from Home)
├── pro/               # Pro/trial state management
├── billing/           # Billing state helpers
├── widget/            # Glance widgets (BatteryWidget, HealthWidget, WidgetDataProvider)
├── worker/            # WorkManager workers (Insights generation and related jobs)
├── service/
│   └── monitor/       # HealthMonitorWorker, HealthMaintenanceWorker,
│                      #   RealTimeMonitorService, ScreenStateTracker,
│                      #   ScreenStateReceiver, MonitoringAlertStateStore,
│                      #   NotificationHelper, MonitorScheduler, BootReceiver
├── di/                # Hilt modules
└── util/              # Logging, timestamp sanitization, live chart buffer
```

## Architecture Conventions

- **Domain layer must stay Android-free in practice** — no `android.*` imports in `domain/`, and avoid `androidx.*` there unless a documented boundary exception exists. Use `String` instead of `android.net.Uri`, map at data/UI boundaries. **Exception:** `androidx.paging.PagingData` is allowed in domain repository interfaces and use cases — wrapping it adds complexity without benefit in a single-module app.
- **UI layer must not import data layer** — ViewModels inject use cases or domain repository interfaces, never data sources or data-layer classes directly.
- **Data layer must not import UI framework** — no Compose types (`ImageBitmap`, etc.) in `data/`. Return platform primitives (`Bitmap`), convert in UI layer.
- **Inject interfaces, not concrete classes** — repositories and data-layer services use interfaces (`DeviceProfileProvider`, `ForegroundAppProvider`). Bind via `@Binds` in Hilt modules.
- **ViewModels must not hold Context** — use `UiText` sealed interface (`UiText.Resource(@StringRes)` / `UiText.Dynamic(value)`) for error messages and status text. Composables resolve with `.resolve()`.
- **Business logic belongs in use cases** — repositories handle data access only. Calculations (e.g., `CalculateFillRateUseCase`), state machines (e.g., `TrackThrottlingEventsUseCase`), and formatting logic go in `domain/usecase/`.
- **`BillingManager` is a lifecycle-aware service, not a repository** — initialized/destroyed by `RuncheckApp`. Widget updates triggered from Application via Flow collection, not from the billing layer.
- **ViewModels must throttle live state flows** — use `.sample(333L)` before `.collect` on combined state flows to limit UI updates to ~3/second and reduce unnecessary recomposition.
- **Debug builds always have Pro enabled** — `BillingManager.initialize()` short-circuits with `_isProState = true` when `BuildConfig.DEBUG` is true, so all Pro features are visible during development.

## Code Style & Conventions

- Write idiomatic Kotlin — use data classes, sealed classes, extension functions
- All UI in Jetpack Compose — no XML layouts, no Fragments
- Use `StateFlow` for ViewModel → UI state
- Use `sealed interface` for UI state (Loading / Success / Error)
- Name ViewModels as `[Screen]ViewModel` (e.g., `HomeViewModel`, `BatteryViewModel`)
- Name UseCases as verb phrases (e.g., `CalculateHealthScoreUseCase`)
- Name composables as nouns (e.g., `ProgressRing`, `GridCard`)
- Keep composables small and focused — extract when > ~50 lines
- All hardcoded strings must go into `strings.xml` for localization
- Comments in English
- No `!!` operator — use safe calls, `requireNotNull`, or sealed error types

## AndroidX Annotations

All new code must use AndroidX annotations where applicable:

- `@RequiresApi(Build.VERSION_CODES.XX)` on any function calling APIs above minSdk (26)
- `@RequiresPermission` on functions that call permission-protected Android APIs
- `@WorkerThread` on functions performing blocking I/O, network, or heavy computation (except Room DAOs and suspend functions)
- `@MainThread` on functions that must run on the UI thread (except Composables)
- `@StringRes`, `@DrawableRes`, `@ColorRes`, `@ColorInt` on Int parameters that represent resources or colors
- `@IntRange` / `@FloatRange` on parameters with known valid ranges (battery %, health score, confidence)
- `@CheckResult` on pure functions where ignoring the return value is a bug

Do not annotate:
- Room DAO methods (Room handles threading)
- Composable functions (inherently main thread)
- Suspend functions (caller controls threading via coroutine dispatcher)
- Private functions only called from one already-annotated place (avoid redundancy)
- Functions that internally check `Build.VERSION.SDK_INT` and handle all API levels with fallbacks — `@RequiresApi` would wrongly prevent callers from using them on lower APIs

## Design System

- **Single dark theme** — no light mode, no AMOLED toggle, no dynamic colors
- **Dark palette:**
  - BgPage = `#0B1E24`, BgCard = `#133040`, BgCardDeep = `#0D2530`, BgCardAlt = `#0F2A35`, BgIconCircle = `#1A3A48`
  - Accent Blue `#4A9EDE` (primary), Accent Teal `#5DE4C7` (secondary), Accent Amber `#E8C44A`
  - Accent Orange `#F5963A`, Accent Red `#F06040`, Accent Lime `#C8E636`, Accent Yellow `#F5D03A`
  - TextPrimary `#E8E8ED`, TextSecondary `#90A8B0`, TextMuted `#7A949E`, TextOnLime `#1A2E0A`
- **Typography:** Manrope (custom, body/headers) + JetBrains Mono (numeric displays) via `MaterialTheme.typography` and `MaterialTheme.numericFontFamily`
- **Navigation:** Push-based from single Home screen (no bottom nav bar), includes Learn section
- **Insights UX:** Home shows a curated subset of up to three insight cards; the full active list lives in a dedicated Insights screen
- **Cards:** Flat backgrounds, no borders, no shadows, no elevation, 16dp rounded corners
- **Tonal layering:** Hero cards use `BgCardDeep` (#0D2530), data cards use `BgCard` (#133040) — creates "recessed instrument panel" depth
- **Hero sections:** Typography-dominant — large 64sp JetBrains Mono values with smaller 28sp units, ProgressRing reduced to 100dp decorative role in Battery/Storage
- **Core components** (34 in `ui/components/` + `ui/components/info/`):
  - Layout: ContentContainer, GridCard (with StatusStrip), ListRow, MetricPill, MetricRow, ActionCard
  - Status: SegmentedStatusBar, StatusStrip (Modifier extension)
  - Indicators: ProgressRing, MiniBar, StatusDot, ConfidenceBadge, SignalBars
  - Charts: TrendChart (oscilloscope sweep, status gradient line, quality zones, tap/drag tooltip), AreaChart (oscilloscope sweep), LiveChart (smooth scroll, glow pulse), HeatStrip, SegmentedBar, SegmentedBarLegend
  - Navigation: PrimaryTopBar, DetailTopBar
  - Typography: AnimatedNumber (AnimatedIntText, AnimatedFloatText), SectionHeader, CardSectionTitle, IconCircle
  - Pro: ProBadgePill, ProFeatureCalloutCard, ProFeatureLockedState
  - Interactive: PullToRefreshWrapper
  - Educational: InfoIcon, InfoBottomSheet, InfoSheetContent (data class), InfoCard (dismissible), CrossLinkButton
- **Section titles:**
  - `SectionHeader` (outline / TextMuted) — page-level sections, above cards
  - `CardSectionTitle` (onSurfaceVariant / TextSecondary) — subsection titles inside cards
- **Corner radii:** Cards = 16dp, small elements (badges, chips) = 8dp
- **Dividers:** `outlineVariant.copy(alpha = 0.35f)` everywhere — no hardcoded colors
- **Icons:** `Icons.Outlined` exclusively — no `Icons.Default`, `Icons.Filled`, or `Icons.Rounded`
- **Spacing:** All padding/spacing values must be on the 4dp grid (2/4/8/12/16/24/32dp)
- **Shared UI metrics:** Touch targets, icon sizes, icon circles, and common CTA heights should come from `UiTokens` rather than being repeated across shared components
- **Animations:** All durations must use `MotionTokens` constants — no bare `tween()` without explicit spec
- **Chart grid alpha:** `outlineVariant.copy(alpha = 0.2f)` for all chart grid lines
- **Value colors:** Data values default to `onSurface`, status labels use `statusColors`. In GridCard, use `statusLabel`/`statusColor` params to separate data from status.
- **Status colors** via `MaterialTheme.statusColors` extension — always paired with icons or text labels for accessibility
  - Healthy = Teal, Fair = Amber, Poor = Orange, Critical = Red
- **Animations:**
  - ProgressRing: 1200ms ease-out (`FastOutSlowInEasing`) from 0 to target
  - MiniBar: 800ms ease-out from 0 to target
  - TrendChart "Instrument Sweep": 3-phase entry (grid fade 200ms → oscilloscope sweep 1000ms with scan line → emphasis 200ms), data transitions (fade-out 300ms → sweep 800ms), `CubicBezierEasing(0.25, 0.1, 0.25, 1)` for sweep
  - TrendChart visual: status gradient line (color follows quality zones), height-proportional fill alpha (peaks 0.30, valleys 0.08), last value emphasis (glow dot + dashed line to Y-axis)
  - AreaChart: same oscilloscope sweep (800ms) + height-proportional fill
  - LiveChart: smooth scroll interpolation (150ms linear) on new data + glow pulse (300ms, radius 8→5dp, alpha 0.5→0.3)
  - All chart animations respect `MaterialTheme.reducedMotion` (instant when true)
  - No card entrance animations
- Contrast ratio minimum: **4.5:1** body text, **3:1** large text (WCAG AA)
- Minimum touch target: 48dp
- Spacing based on 4dp grid: 4/8/12/16/24/32dp tokens via `MaterialTheme.spacing`
- **Adaptive layout:** `ContentContainer` wrapper constrains content to max 600dp width, centered — applied to all detail screens. TopBar remains full-width. Home screen grid uses 1×4 row on wide screens (≥600dp), 2×2 on phones. FullscreenChartScreen forces landscape orientation.

## Battery Features

The battery detail screen has extensive monitoring capabilities:

- **Hero card:** ProgressRing with level %, status text, optional mAh remaining (`BATTERY_PROPERTY_CHARGE_COUNTER`)
- **W + mV display:** Power in watts and voltage in millivolts shown under the mA current reading
- **Current stats:** In-memory avg/min/max tracking that resets when charging status changes
- **Battery capacity:** Design capacity via PowerProfile reflection, estimated capacity from `designCapacity × healthPercent / 100`
- **Screen On/Off tracking:** `ScreenStateTracker` with BroadcastReceiver for `ACTION_SCREEN_ON/OFF/POWER_DISCONNECTED`, tracks drain rate per screen state
- **Deep Sleep / Held Awake:** Tracks `PowerManager.isDeviceIdleMode` during screen-off periods
- **Long-term statistics:** `GetBatteryStatisticsUseCase` queries Room for charged/discharged totals, session counts, average drain rates over configurable period
- **History:** TrendChart with `SINCE_UNPLUG`, Day, Week, Month, All periods. SINCE_UNPLUG queries last charging timestamp from Room. Y-axis labels, X-axis time labels, grid, tooltip (value + timestamp), quality zones for Level and Temperature, Min/Avg/Max MetricPills.
- **Session graph:** Current and power charts with 15m/30m/All windows during charging. Y-axis, X-axis, grid, tooltip, Min/Avg/Max stats.

## Storage Features

The storage detail screen provides monitoring and cleanup tools:

- **Hero card:** ProgressRing with usage %, used/total, fill rate, cache total, free space MetricPills
- **Media breakdown:** `SegmentedBar` (Canvas, 12dp, animated) with 6 categories: Images (Teal), Videos (Blue), Audio (Orange), Documents (Lime), Downloads (Yellow), Other (Muted). `SegmentedBarLegend` with StatusDots.
- **Cleanup tools:** `ActionCard` components (outlined border) linking to reusable `CleanupScreen`:
  - Large Files (threshold: > 10/50/100/500 MB)
  - Old Downloads (age: > 30/60/90/365 days)
  - APK Files (no filter, pre-selected) — scans both Downloads + Files collections with filename + MIME type matching
  - Trash (API 30+, empty via `MediaStore.createDeleteRequest` with `getTrashedUris()` + ActivityResultLauncher)
- **Cleanup screen:** Shared `cleanup/{type}` route, category-grouped file list with thumbnails, `MiniBar` per file, `CleanupBottomBar` with before/after projection, `CleanupSuccessOverlay` fade animation
- **Delete mechanism:** API 30+ `createDeleteRequest` → system dialog via `ActivityResultLauncher`; API 29 `ContentResolver.delete` fallback
- **Fill rate:** `CalculateFillRateUseCase` — linear regression on Room history (7-day lookback)
- **Details card:** Total/Used/Available/Apps/Cache + technical details (File System from `/proc/mounts`, Encryption from `ro.crypto.type` system property, Storage Volumes from `StorageManager`)
- **SD card:** Shown if detected via `getExternalFilesDirs`
- **Permissions:** `READ_MEDIA_IMAGES`, `READ_MEDIA_VIDEO`, `READ_MEDIA_AUDIO` (API 33+) + `READ_EXTERNAL_STORAGE` (API ≤32) — runtime-requested on StorageDetailScreen entry

## Insights Features

- **Persisted insights:** Room-backed insight rows generated from historical device data rather than transient UI-only suggestions
- **Generation:** `InsightGenerationWorker` runs on the monitoring scheduler lifecycle and refreshes insight rows in the background
- **Home preview:** Home shows a curated subset of up to three insights to reduce noise
- **Dedicated screen:** A separate Insights screen exposes the full active list, unseen count behavior, dismiss actions, and destination deep links
- **Ranking policy:** Home preview prefers one strong insight per destination area before filling remaining slots
- **Release safety:** The default `InsightDebugActions` implementation is a release-safe no-op; debug builds override it with deterministic seeding/regeneration tools

## Settings Features

Settings screen uses grouped card layout with these sections:

- **Monitoring:** Interval selection (15/30/60 min)
- **Live Notification:** Opt-in persistent notification with real-time battery stats. Master toggle starts/stops `RealTimeMonitorService`. Per-metric toggles: Current (mA/W), Charging status, Temperature, Screen stats, Remaining time. Sub-toggles only visible when master is enabled. Disabled by default.
- **Notifications:** Master toggle + per-alert toggles (Low Battery, High Temp, Low Storage, Charge Complete). Master off dims and disables sub-toggles.
- **Alert thresholds:** Sliders for battery (5–50%, default 20), temperature (35–50°C, default 42), storage (70–99%, default 90). Value displayed in numericFontFamily with primary color.
- **Display:** Temperature unit (°C/°F) — stored in DataStore
- **Data:** Retention (Pro), export (CSV), clear speed tests, clear all data — all destructive actions require confirmation dialog
- **Pro:** Status display, purchase button, restore purchase
- **Device:** Read-only MetricPill grid (model, API level, current reliability, cycle count, thermal zones)
- **About:** Version, Rate on Play Store, Privacy Policy, Send Feedback

All preferences stored in `DataStore<Preferences>` via `UserPreferencesRepository`.

## Educational Content System

Three-tier in-app educational system explaining technical metrics and concepts to users:

### Tier 1 — Info Bottom Sheets (per metric)
- Small `(?)` icon (`InfoIcon`, 16dp, 48dp touch target) next to metric labels in `MetricRow` and `MetricPill` via optional `onInfoClick` parameter
- Tapping opens `InfoBottomSheet` (`ModalBottomSheet`, max 60% screen height, scrollable) with: title, plain-language explanation, "What's normal" highlight card (`surfaceContainerHigh`, 8dp corners), "Why it matters", optional expandable "Learn more" deeper detail (`AnimatedVisibility`)
- Content defined in `*InfoContent.kt` objects per screen (e.g., `BatteryInfoContent.voltage`)
- State: `var activeInfoSheet by rememberSaveable { mutableStateOf<String?>(null) }` in each detail screen content composable
- **Coverage:** Battery (21 metrics), Thermal (4), Network (13), Storage (6), Speed Test (4), Settings (3) = 51 total

### Tier 2 — Contextual Info Cards (per screen section)
- `InfoCard` composable: `surfaceContainerHigh` background, 3dp left `primary` accent border (`Modifier.drawBehind`), `InfoOutlined` 20dp icon, `Close` dismiss button, 16dp corners
- Dismissible — dismissed state stored in DataStore via `UserPreferencesRepository.getDismissedInfoCards()` / `dismissInfoCard(id)` using `stringSetPreferencesKey`
- Card IDs defined in `*InfoCards.kt` objects (e.g., `BatteryInfoCards.HEALTH_80_PERCENT`)
- Each ViewModel collects `dismissedInfoCards: Set<String>` into UiState and exposes `dismissInfoCard(id: String)`
- Cards shown conditionally (e.g., health < 90%, charging, high drain, full storage) and disappear permanently when dismissed
- **Coverage:** Battery (5 cards), Thermal (2), Network (2), Storage (2) = 11 total

### Tier 3 — Learn Section (standalone screen)
- Accessible from Home screen Quick Tools card (Learn `ListRow` with `MenuBook` icon)
- `LearnScreen`: scrollable list of `LearnArticleCard` composables grouped by `LearnTopic` (Battery, Temperature, Network, Storage, General) with `CardSectionTitle` per group
- `LearnArticleDetailScreen`: body text parsed with `## ` headers → `titleSmall`, `\n\n` paragraph breaks → `bodyMedium`, `CrossLinkButton` for navigating to relevant detail screens
- Article catalog: `LearnArticleCatalog` object with 15 articles (Battery 4, Temperature 3, Network 3, Storage 2, General 3)
- Navigation: `Screen.Learn` + `Screen.LearnArticle(articleId)` routes in NavGraph
- **Not Pro-gated** — all educational content is free

## Device Detection System

The `DeviceCapabilityManager` determines what data is reliably available on the current device. This is critical — the app must NEVER show inaccurate data without warning.

Rules:
- Always validate `BATTERY_PROPERTY_CURRENT_NOW` at startup — check for non-zero, changing, plausible values
- Store results in `DeviceProfile` and use it throughout the app
- Show confidence badges: green "Accurate", yellow "Estimated", gray "Unavailable"
- If a metric is unavailable, hide it or show "Not supported on this device" — never show 0 or garbage values

## Manufacturer-Specific Handling

Use `BatteryDataSourceFactory` to select the best data source based on device:
- API 34+ devices: use new `BATTERY_PROPERTY_CHARGING_CYCLE_COUNT` and `STATE_OF_HEALTH`
- Samsung: handle potential max-theoretical-current-only readings
- OnePlus: handle SUPERVOOC sign conventions
- Google Pixel: typically most reliable, use as baseline
- Fallback: `GenericBatterySource` with confidence warnings

## Background Monitoring

- Use WorkManager for periodic background readings (not foreground service for periodic work)
- Default interval: 30 minutes (user-configurable: 15 / 30 / 60 min)
- Foreground service (`RealTimeMonitorService`) for two purposes:
  1. When user is actively viewing real-time data on Battery screen (binding-based lifecycle)
  2. When user enables **Live Notification** in Settings (opt-in, persistent until disabled)
- Live notification mode: `BigTextStyle`, updates every 5s, configurable metrics, stays active even when app is closed
- `ScreenStateTracker` runs as `@Singleton`, started/stopped by BatteryViewModel lifecycle
- Respect battery optimization — don't fight the OS
- Detect and handle data gaps gracefully in trend charts

## Database

- Room with auto-migrations
- Tables: `battery_readings`, `storage_readings`, `network_readings`, `thermal_readings`, `throttling_events`, `charging_sessions`, `charger_profiles`, `speed_test_results`, `app_battery_usage`, `devices`
- Free tier: retain only 24 hours of readings (delete older on each write)
- Pro tier: configurable retention (3mo / 6mo / 1yr / forever)
- Indices on timestamp columns for efficient range queries

## Testing

- Write unit tests for all UseCases and scoring logic
- Write unit tests for DeviceProfile validation logic
- UI tests with Compose testing framework for critical flows
- Use fakes/mocks for system APIs (BatteryManager etc.) in tests

## Monetization

- Free version: core monitoring with locked Pro feature entry points where applicable
- Pro version: one-time in-app purchase (€3.49), unlocks extended history, widgets, charger comparison, export, advanced insights, and storage cleanup tools
- Trial system with expiration modal and notification worker
- Use Google Play Billing Library
- Gate pro features with `BillingManager` (implements `ProStatusProvider` + `ProPurchaseManager`)
- Ad banners on detail screens (free tier only)

## Build & Release

- Use a single `app` module (no multi-module until necessary)
- **Static analysis:** ktlint (formatting) + detekt 1.23.8 with compose-rules 0.4.27 (code quality, `ignoreFailures = true` during adoption) + Android Lint (correctness/security/a11y checks)
- ProGuard/R8 minification enabled for release builds
- Generate signed APK/AAB for Play Store
- Version code: auto-increment
- Version name: semver (1.0.0)

## Build Notes

- AGP 9.1.0 built-in Kotlin handles Kotlin compilation; `kotlin.compose` plugin applied separately for Compose compiler support
- `android.disallowKotlinSourceSets=false` is needed for KSP generated sources with AGP 9
- `BatteryManager.BATTERY_PROPERTY_CHARGING_CYCLE_COUNT` and `STATE_OF_HEALTH` are not in the public SDK — use raw integer constants (8 and 12)
- Pull-to-refresh uses `PullToRefreshBox` (not the deprecated `PullToRefreshContainer`)

## Important Notes

- This is an Android-only app — no iOS, no cross-platform
- Privacy-first: no analytics, no tracking, no account system, no crash reporting, no network calls except latency ping and speed test
- All measurement and history data stays on device — zero data leaves the phone
- The roadmap and next steps are documented in `docs/plans/2026-03-10-phase1-completion-and-roadmap.md`

## Feature Specs

- `docs/battery-enhancements-spec.md` — Battery & thermal enhancements (mAh remaining, W+mV, current stats, temp min/max, screen on/off, sleep analysis, statistics, since-unplug history)
- `docs/storage-enhancements-spec.md` — Storage feature expansion (media breakdown, top apps, cleanup tools, trash management, large file scanner, history chart)
- `docs/storage-ui-design.md` — Storage UI design spec (SegmentedBar, ActionCard, FileListItem, visual patterns)
- `docs/storage-cleanup-spec.md` — Storage cleanup implementation (CleanupScreen, delete flow, thumbnails, category groups, success overlay)
- `docs/settings-enhancements-spec.md` — Settings enhancements (per-alert notifications, alert thresholds, temperature unit, data management, grouped layout)
- `docs/info-tooltips-spec.md` — Info tooltip system (superseded by `educational-content-system.md`)
- `docs/superpowers/specs/2026-03-24-chart-animation-design.md` — Chart animation "Instrument Sweep" design spec (oscilloscope sweep, status gradient line, improved fill, LiveChart smooth scroll)
- `docs/ui-consistency-audit.md` — UI consistency findings and fixes
- `docs/ui-reference.md` — UI component reference and patterns
- `docs/release-checklist.md` — Release build and Play Store checklist
- `docs/play-store-listing.md` — Play Store listing content
- `docs/privacy-policy.md` — Privacy policy
- `educational-content-system.md` — Three-tier educational content system spec (info bottom sheets, contextual cards, Learn section)

## Security & Quality Requirements

This project is published on Google Play and handles device hardware data. All code must follow these security and quality rules. Studies show AI-generated code contains security vulnerabilities in 45% of cases, with Java/Kotlin having the highest failure rate (70%+). These rules exist to prevent the most common AI coding mistakes.

### Mandatory Security Rules

#### PendingIntents
- EVERY PendingIntent MUST use an explicit Intent with a target component class: `Intent(context, TargetClass::class.java)`
- NEVER create PendingIntents with implicit Intents (action-only, no component)
- `setPackage()` alone is NOT sufficient — always set the component explicitly
- ALWAYS include FLAG_IMMUTABLE unless mutability is specifically required

#### Data Storage
- NEVER store sensitive data in plain SharedPreferences — use EncryptedSharedPreferences or Android Keystore
- NEVER store API keys, tokens, or secrets in source code, strings.xml, or BuildConfig
- Set `android:allowBackup="false"` in AndroidManifest.xml for release builds
- NEVER log sensitive data with Log.d/Log.i/Log.w/Log.e

#### Network Security
- ALL network calls MUST use HTTPS
- Maintain network_security_config.xml with cleartextTrafficPermitted="false"
- NEVER disable certificate validation or create TrustManagers that accept all certificates

#### AndroidManifest
- EVERY activity, service, receiver, and provider MUST have explicit `android:exported` attribute
- Do NOT export components unless specifically required
- Declare minimum necessary permissions only
- Use foreground service types for all foreground services

#### Cryptography
- NEVER use MD5 or SHA1 for security purposes
- NEVER use DES or ECB mode encryption
- NEVER hardcode encryption keys or initialization vectors
- Use Android Keystore for key storage

### Mandatory Code Quality Rules

#### Null Safety
- NEVER use `!!` (force unwrap) — always use safe alternatives: `?.`, `?:`, `let`, `requireNotNull` with meaningful message
- If you think `!!` is needed, restructure the code to eliminate the nullable type

#### Lifecycle & Memory
- NEVER store Activity or Fragment context in ViewModel, singleton, companion object, or any long-lived object — use Application context if needed
- ALL coroutines MUST be scoped to viewModelScope or lifecycleScope — NEVER use GlobalScope
- ALL observers, listeners, callbacks, and BroadcastReceivers MUST be unregistered in the matching lifecycle method
- ALL Closeable resources MUST use .use {} blocks
- NEVER load full-size bitmaps — always specify target dimensions

#### Error Handling
- NEVER leave catch blocks empty — at minimum log the exception
- NEVER catch Exception broadly when a specific exception type is appropriate
- ALL launch {} blocks that can fail MUST have error handling (try/catch or CoroutineExceptionHandler)

#### Intent & Component Security
- VALIDATE all Intent extras before use — never assume they exist or have the expected type
- VALIDATE all ContentProvider query parameters
- CHECK permissions programmatically before accessing protected resources

#### Build Configuration
- ProGuard/R8 MUST be enabled for release builds
- `debuggable` MUST be false in release buildType
- Target the latest stable Android SDK

### Before Every PR

Before creating a pull request, verify:
1. `./gradlew assembleDebug` compiles without errors
2. No `!!` operators added
3. No hardcoded secrets or API keys
4. No new exported components without justification
5. All PendingIntents use explicit Intents with FLAG_IMMUTABLE
6. All coroutines properly scoped to lifecycle
7. No empty catch blocks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Insaner1980) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
