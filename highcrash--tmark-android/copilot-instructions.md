## tmark-android

> Android client for the **T-Mark** equipment rental platform. Users browse camera/lighting/audio packages, build rental requests, track orders, view invoices, and manage their profile.

# CLAUDE.md — T-Mark Android Client

## Project Overview
Android client for the **T-Mark** equipment rental platform. Users browse camera/lighting/audio packages, build rental requests, track orders, view invoices, and manage their profile.

**Backend**: Next.js at `https://tmark.online/api/mobile/` (source: `D:\T-MarkGITHUB\tmark`)

## Key Documentation
- **`API_REFERENCE.md`** — Complete API docs (48 endpoints, request/response schemas, error codes)
- **`design-reference/DESIGN_SYSTEM.md`** — Full design system (colors, typography, spacing, components)
- **`design-reference/visual-reference.html`** — Interactive visual reference (open in browser)
- **`design-reference/app-icon/`** — App icon source files + Play Store icon
- **`design-reference/logo/`** — Logo on black/white backgrounds

## Architecture
- **Pattern**: Single-Activity MVVM with Jetpack Compose
- **DI**: Hilt (`@HiltAndroidApp`, `@AndroidEntryPoint`, `@HiltViewModel`)
- **Networking**: Retrofit 2.11 + Moshi 1.15 + OkHttp 4.12
- **State**: `StateFlow` in ViewModels, `collectAsState()` in Composables
- **Auth**: Bearer token stored in DataStore, injected via OkHttp interceptor
- **Navigation**: Compose Navigation with sealed `Screen` class routes

## Project Structure
```
app/src/main/java/com/tmark/client/
├── TMarkApp.kt              # Hilt Application
├── MainActivity.kt           # Single Activity, edge-to-edge, splash screen
├── SmsRetrieverReceiver.kt   # SMS OTP auto-read
├── data/
│   ├── api/
│   │   ├── ApiService.kt     # All Retrofit endpoints
│   │   └── ApiResult.kt      # Sealed result wrapper + safeApiCall
│   ├── local/
│   │   └── TokenStore.kt     # DataStore for auth tokens
│   ├── model/                # Moshi @JsonClass data classes
│   │   ├── AuthModels.kt     # OTP, register, email login
│   │   ├── CatalogModels.kt  # Packages, items, sub-packages
│   │   ├── CartModels.kt     # SelectedEquipment
│   │   ├── RequestModels.kt  # Rental requests, bootstrap
│   │   ├── OrderModels.kt    # Orders + details
│   │   ├── InvoiceModels.kt  # Invoices
│   │   ├── DashboardModels.kt# Stats
│   │   └── ProfileModels.kt  # Profile + designation
│   ├── repository/           # @Singleton repos wrapping safeApiCall
│   └── CartManager.kt        # In-memory cart state
├── di/
│   └── NetworkModule.kt      # Hilt module: Moshi, OkHttp, Retrofit, ApiService
├── navigation/
│   ├── Screen.kt             # Sealed class routes
│   └── AppNavigation.kt      # NavHost + 5-tab bottom nav
└── ui/
    ├── components/
    │   └── TMarkComponents.kt # ScreenHeader, StatusBadge, TMarkButton, TextField, etc.
    ├── screens/               # Feature screens (auth, catalog, cart, dashboard, orders, invoices, profile, requests)
    └── theme/
        ├── Color.kt           # TMarkRed #D42B1E, TMarkBlack #0A0908, status colors
        ├── Type.kt            # BebasNeue, BarlowCondensed, Barlow fonts
        └── Theme.kt           # Material3 color scheme
```

## Design System
- **Primary**: `TMarkRed` (#D42B1E), `TMarkBlack` (#0A0908), `TMarkOffWhite` (#F5F4F2)
- **Muted**: `TMarkMuted` (#888582), `TMarkBorder` (#E5E4E2)
- **Fonts**: Bebas Neue (headings), Barlow Condensed (labels/eyebrows), Barlow (body)
- **Edge-to-edge**: All screens use `statusBarsPadding()` / `navigationBarsPadding()`
- **Headers**: Dark background with embedded back nav (no separate ScreenHeader + content band)
- **Pattern**: Red accent lines/dots for visual hierarchy, uppercase labels with wide letter-spacing

## Conventions
- All API models use `@JsonClass(generateAdapter = true)` with `@Json(name = "...")` annotations
- ViewModels expose `StateFlow<UiState>` — screens collect with `collectAsState()`
- Repositories are `@Singleton @Inject constructor` — always wrap calls in `safeApiCall`
- Navigation args go in the `Screen` sealed class route strings
- Bottom tabs hide on detail/form screens (listed in `noTabRoutes`)
- Phone numbers: Bangladeshi format, validated with `^01[3-9]\d{8}$`
- Currency: Bangladeshi Taka (৳), formatted with `%,.0f`

## Build
```bash
./gradlew assembleDebug        # Debug APK
./gradlew assembleRelease      # Release APK
./gradlew installDebug         # Install on connected device
```

- **compileSdk**: 36, **minSdk**: 26, **targetSdk**: 36
- **Kotlin**: 2.0.21, **Compose BOM**: 2025.05.00, **Hilt**: 2.51.1
- API base URL configured in `buildConfigField` in `app/build.gradle.kts`

## Key Decisions
- Cart is in-memory (`CartManager`) — not persisted across app restarts (by design)
- Auth flow: Phone OTP (primary) → Email login (secondary) → Registration (new users)
- Sub-packages (combo packages) render as separate sections with category-grouped items
- SMS Retriever API used for auto-OTP (no SMS permission needed)
- No image caching strategy yet — Coil handles it with defaults

## Adding a New Screen
1. Add data model in `data/model/`
2. Add API endpoint in `ApiService.kt`
3. Add repository method in appropriate `data/repository/`
4. Create ViewModel in `ui/screens/{feature}/`
5. Create Screen composable in same folder
6. Add route to `Screen.kt` sealed class
7. Add `composable()` entry in `AppNavigation.kt`
8. If detail/form screen, add route to `noTabRoutes`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/highcrash)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/highcrash)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
