## pet-circle

> Flutter app for collaborative canine respiratory monitoring (Sleeping Respiratory Rate / SRR).

# Pet Circle — Claude Code Instructions

## Project Overview

Flutter app for collaborative canine respiratory monitoring (Sleeping Respiratory Rate / SRR).
Roles: Pet Owner and Veterinarian at the app level; Admin / Member / Viewer per-pet via care circles.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Dart / Flutter (SDK ^3.10.4) |
| Backend | Firebase (Auth, Firestore, Deep Links) |
| Auth | Firebase Auth + Google Sign-In + Sign in with Apple |
| Charts | Syncfusion Flutter Charts |
| Notifications | flutter_local_notifications + timezone |
| Localisation | flutter_localizations — `en` and `he` supported |
| State | `ChangeNotifier` stores (global singletons) |

## Architecture

**Global store singletons** — all live in `lib/stores/`:
- `petStore` — pets, care circle membership, access control
- `measurementStore`, `medicationStore`, `noteStore` — per-pet Firestore subscriptions
- `userStore`, `settingsStore`, `notificationStore` — user/app-level state

Stores expose `subscribeForUser(uid)` / `cancelSubscription()` Firestore streams.
When `kEnableFirebase = false` (in `lib/main.dart`), stores are seeded from `lib/data/mock_data.dart` instead.

**Services** (`lib/services/`) — thin Firestore/Firebase wrappers.
**Models** (`lib/models/`) — immutable data classes with `copyWith`.
**Screens** (`lib/screens/`) — organised by feature: `auth/`, `dashboard/`, `measurement/`, etc.
**Widgets** (`lib/widgets/`) — shared UI: `NeumorphicCard`, `PrimaryButton`, `StatusBadge`, etc.

## Key Entry Points

- **App entry**: `lib/main.dart` → `PetCircleApp`
- **Routes**: `lib/app_routes.dart`
- **Auth gate**: `lib/screens/auth/auth_gate.dart`
- **Theme**: `lib/theme/app_theme.dart`
- **Localisation**: `lib/l10n/` (generated from ARB files via `flutter gen-l10n`)

## Code Conventions

- Models are **immutable** — always use `copyWith`, never mutate in-place.
- Stores mutate their own private lists then call `notifyListeners()`.
- All user-visible strings go through `AppLocalizations` (no hardcoded EN strings in widgets).
- File naming: `snake_case.dart` for all Dart files.
- Feature flag: `const bool kEnableFirebase = true` in `main.dart` — toggle for mock-data dev.

## Common Commands

<!-- AUTO-GENERATED from pubspec.yaml and firebase-status.md -->
| Command | Description |
|---------|-------------|
| `flutter run` | Run on connected device / simulator |
| `flutter test` | Run all tests |
| `flutter test --coverage` | Run tests with coverage report |
| `flutter analyze` | Lint and static analysis |
| `flutter gen-l10n` | Regenerate localisation files from ARB |
| `flutter pub get` | Install/update dependencies |
| `flutter pub outdated` | Check for outdated packages |
| `flutter build apk` | Build Android release APK |
| `flutter build ios` | Build iOS release |
| `flutterfire configure --project=pet-circle-app` | Regenerate Firebase config files |
| `firebase deploy --only firestore:rules --project pet-circle-app` | Deploy Firestore security rules |
<!-- END AUTO-GENERATED -->

## Project Structure

```
lib/
  main.dart            # App entry, kEnableFirebase flag, mock seeding
  app_routes.dart      # Named route constants
  firebase_options.dart
  data/mock_data.dart  # Dev-mode seed data
  models/              # Immutable data classes
  stores/              # ChangeNotifier global singletons
  services/            # Firestore / Firebase service layer
  screens/             # Feature screens (auth, dashboard, measurement, …)
  widgets/             # Shared UI components
  theme/               # AppTheme, AppAssets
  l10n/                # Localisation (en + he)
assets/figma/          # Design assets
docs/                  # PRD, bug log, firebase status, future features
```

## Where to Look

| Task | Location |
|------|---------|
| Add a screen | `lib/screens/<feature>/`, register in `lib/app_routes.dart` + `main.dart` |
| Add a model field | `lib/models/<model>.dart` — add to constructor + `copyWith` |
| Add Firestore logic | `lib/services/pet_service.dart` or relevant service |
| Change store state | `lib/stores/<store>.dart` — mutate private field, call `notifyListeners()` |
| Add a localised string | `lib/l10n/app_en.arb` + `app_he.arb`, then `flutter gen-l10n` |
| Track a bug | `docs/bug-log.md` |
| Review future features | `docs/future-features.md` |

## Current Status

- **Phase:** Phase 2 — active feature development (Phase 1 complete: Firebase wiring, auth, stores, basic screens)
- **Known bugs:** See `docs/bug-log.md`
- **Planned features:** See `docs/future-features.md`

### Important constraints
- Do not change Firestore document schema without also updating `firestore.rules`
- `kEnableFirebase = true` in production; use `false` only for widget test dev
- All new user-visible strings require entries in both `app_en.arb` and `app_he.arb`

## Testing

- Test files live in `test/` matching the source path.
- Run with `flutter test`.
- When `kEnableFirebase = false`, all stores work from mock data — useful for widget tests.
- Shared test helpers in `test/helpers/` (`test_app.dart`, `mock_stores.dart`).
- Coverage target: **80%+** line coverage.

---

## Figma Design System Rules

### Figma Sources

- **Design system**: https://www.figma.com/design/ApTk87wJXejOTzVtEnFJMw/Pet-circle?node-id=264-1093
- **Views**: https://www.figma.com/design/ApTk87wJXejOTzVtEnFJMw/Pet-circle?node-id=167-107
- **Conflict rule**: If design system and views disagree, **follow the views**.
- **Migration mapping**: See `docs/design-system-migration.md` for complete token tables.

### Required Figma-to-Code Flow

1. Run `get_design_context` with the Figma node for the component/screen
2. Run `get_screenshot` for a visual reference
3. Download any assets from the Figma MCP asset endpoint
4. Translate Figma output (React + Tailwind) into Flutter/Dart using project conventions
5. Use the project's semantic token classes — **never hardcode hex values**
6. Validate against the Figma screenshot for 1:1 visual parity

### Design Token Architecture (3 layers)

```
lib/theme/
  tokens/
    colors.dart       # Primitive palette (AppPrimitives) — raw hex values
    typography.dart   # Font definitions (AppTypography) — sizes, weights, line heights
    spacing.dart      # Spacing + radius scales (AppSpacingTokens, AppRadiiTokens)
    shadows.dart      # Shadow definitions (AppShadowTokens)
  semantic/
    color_scheme.dart # Semantic ThemeExtension (AppSemanticColors) — light + dark
    text_theme.dart   # Semantic text styles (AppSemanticTextStyles)
  app_theme.dart      # buildAppTheme() / buildDarkTheme() — wires everything
  app_assets.dart     # Asset path registry
```

### Color System

**Font:** Instrument Sans (variable font, `fontVariationSettings: "'wdth' 100"`)

**Palette categories:** Ink (grays), Sky (backgrounds), Primary (purple #6B4EFF), Red, Green, Yellow, Blue.
Each has 5 scales: Lightest, Lighter, Light, Base, Dark/Darkest.

**IMPORTANT rules:**
- NEVER hardcode hex colors in widgets or screens — always use semantic tokens
- Access colors via `AppSemanticColors.of(context).primary` (ThemeExtension pattern)
- Primary actions use `Primary/Base` (#6B4EFF) — NOT the old chocolate color
- Text colors use `Ink/*` tokens — NOT hardcoded black
- Background colors use `Sky/*` or `Primary/Lightest` — NOT hardcoded white

### Typography System

**Font family:** Instrument Sans (replaces Inter)
**5 size categories:** Title 1 (48px), Title 2 (32px), Title 3 (24px), Large (18px), Regular (16px), Small (14px), Tiny (12px)
**3 line-height variants per size:** None (= size), Tight, Normal
**3 weights:** Bold (700), Medium (500), Regular (400)

Access via `AppSemanticTextStyles.title1`, `AppSemanticTextStyles.largeBold`, etc.

### Shadow System

3 elevation levels (replaces neumorphic shadows):
- `Shadow/Small` — subtle, for cards and containers
- `Shadow/Medium` — moderate, for elevated elements
- `Shadow/Large` — prominent, for modals and overlays

### Component Conventions

- Buttons: pill-shaped (`borderRadius: 48`), filled (Primary/Base bg) or outlined (Primary/Base border)
- Cards: `borderRadius: 16`, `Primary/Lightest` bg or white, Shadow/Small
- Icon buttons: circular (`borderRadius: 1000`), `Sky/Light` bg
- Tab bar: **5 tabs** (Home, Trends, Diary, Measure, Medicine), active = Primary/Base
- Inputs: `borderRadius: 16`, Sky/Lighter fill
- Avatars: circular, 32px (small) or 64px (large)

### Asset Handling

- IMPORTANT: If Figma MCP returns a localhost source for images/SVGs, use that directly
- DO NOT add new icon packages — all assets come from Figma payload
- Store downloaded assets in `assets/figma/`

### What NOT to Do

- NEVER use the old `AppColorsTheme.of(context)` — use `AppSemanticColors.of(context)`
- NEVER use the old `AppTextStyles.*` — use `AppSemanticTextStyles.*`
- NEVER use neumorphic shadows — use `AppShadowTokens.*`
- NEVER hardcode the old color values (chocolate, pink, cherry, offWhite, etc.)
- NEVER skip the Figma screenshot comparison step

---
> Source: [HiLuLiT/pet-circle](https://github.com/HiLuLiT/pet-circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
