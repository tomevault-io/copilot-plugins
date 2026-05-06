## figma-design-system

> Rules for implementing Figma designs using the Figma MCP server. Covers component organization, styling conventions, design tokens, asset handling, and the required Figma-to-code workflow for this Flutter/Dart pet health monitoring app.


# Pet Circle — Figma Design System Rules

## Project Overview

Pet Circle is a **Flutter/Dart** cross-platform app (iOS, Android, Web, Desktop) for pet health monitoring. It uses a **neumorphic design system** with centralized design tokens aligned to Figma variables. Typography is powered by **Google Fonts (Inter)**.

## Figma MCP Integration Rules

These rules define how to translate Figma inputs into code for this project and must be followed for every Figma-driven change.

### Required Flow (do not skip)

1. Run `get_design_context` first to fetch the structured representation for the exact node(s)
2. If the response is too large or truncated, run `get_metadata` to get the high-level node map, then re-fetch only the required node(s) with `get_design_context`
3. Run `get_screenshot` for a visual reference of the node variant being implemented
4. Only after you have both `get_design_context` and `get_screenshot`, download any assets needed and start implementation
5. Translate the output (usually React + Tailwind) into **Flutter widgets and Dart code** using this project's conventions, tokens, and architecture
6. Validate against Figma for 1:1 look and behavior before marking complete

### Implementation Rules

- IMPORTANT: Treat the Figma MCP output (React + Tailwind) as a **representation of design and behavior**, not as final code
- IMPORTANT: Translate all React components to **Flutter StatelessWidget / StatefulWidget** classes
- IMPORTANT: Replace Tailwind utility classes with the project's design tokens (`AppColors`, `AppSpacing`, `AppRadii`, `AppTextStyles`, `AppShadows`)
- Reuse existing widgets from `lib/widgets/` instead of duplicating functionality
- Use the project's color system, typography scale, and spacing tokens consistently
- Respect existing routing (`app_routes.dart`), state management, and data-fetch patterns
- Strive for 1:1 visual parity with the Figma design
- Validate the final UI against the Figma screenshot for both look and behavior

---

## Component Organization

### Directory Structure

```
lib/
├── widgets/          # Reusable UI components (buttons, cards, inputs, etc.)
├── screens/          # Screen-level components, organized by feature
│   ├── auth/
│   ├── dashboard/
│   ├── measurement/
│   ├── medication/
│   ├── messages/
│   ├── onboarding/
│   ├── pet_detail/
│   ├── settings/
│   └── trends/
├── models/           # Data models (Pet, User, Measurement, Medication, etc.)
├── stores/           # ChangeNotifier stores (pet, measurement, note, medication, etc.)
├── services/         # Business logic (auth_service, user_service)
├── providers/        # State providers
├── theme/            # Design tokens and theme configuration
│   ├── app_theme.dart    # Colors, spacing, typography, shadows, radii
│   └── app_assets.dart   # Asset path constants
├── data/             # Mock/demo data
└── l10n/             # Localization files (English, Hebrew)
```

### Placement Rules

- IMPORTANT: Place new **reusable UI components** in `lib/widgets/`
- IMPORTANT: Place new **screen components** in `lib/screens/<feature>/`
- Place new **data models** in `lib/models/`
- Place new **services** in `lib/services/`
- Place new **asset constants** in `lib/theme/app_assets.dart`

### Existing Widgets (check before creating new ones)

| Widget | File | Purpose |
|--------|------|---------|
| `PrimaryButton` | `lib/widgets/primary_button.dart` | Full-width button (filled/outlined variants) |
| `RoundIconButton` | `lib/widgets/round_icon_button.dart` | Circular icon button |
| `NeumorphicCard` | `lib/widgets/neumorphic_card.dart` | Card with neumorphic shadows |
| `LabeledTextField` | `lib/widgets/labeled_text_field.dart` | Text field with label |
| `BreedSearchField` | `lib/widgets/breed_search_field.dart` | Searchable breed dropdown (148 breeds, live filter) |
| `AppDropdown` | `lib/widgets/app_dropdown.dart` | Label + dropdown selector with chevron |
| `SettingsRow` | `lib/widgets/settings_row.dart` | Settings row (icon + title + trailing) |
| `AppHeader` | `lib/widgets/app_header.dart` | Header with avatar, pet switcher chip, notification bell |
| `BottomNavBar` | `lib/widgets/bottom_nav_bar.dart` | Tab bar: Home, Trends, Measure, Medication |
| `StatusBadge` | `lib/widgets/status_badge.dart` | Colored status badge |
| `TogglePill` | `lib/widgets/toggle_pill.dart` | Pill-shaped toggle |
| `UserAvatar` | `lib/widgets/user_avatar.dart` | User avatar with fallback |
| `DogPhoto` | `lib/widgets/dog_photo.dart` | Pet photo with fallback |
| `AppImage` | `lib/widgets/app_image.dart` | Image with error handling |
| `OnboardingShell` | `lib/widgets/onboarding_shell.dart` | Onboarding layout with Back/Next buttons |

---

## Design Tokens

All tokens are centralized in `lib/theme/app_theme.dart`. IMPORTANT: Never hardcode colors, spacing, or typography values — always use the token classes.

### Colors — `AppColors` (9 Figma primitive tokens)

```dart
AppColors.white       // #FFFFFF — backgrounds, card surfaces
AppColors.offWhite    // #F8F1E7 — warm background, scaffold
AppColors.lightYellow // #FFECB7 — accent highlights
AppColors.chocolate   // #402A24 — primary text, buttons
AppColors.pink        // #FFC2B5 — soft accent, decorative
AppColors.cherry      // #E64E60 — alerts, warnings, errors
AppColors.lightBlue   // #75ACFF — info, secondary accent
AppColors.blue        // #146FD9 — links, interactive elements
AppColors.black       // #000000 — rare, inverted contexts
```

- IMPORTANT: **Never hardcode hex colors.** Always use `AppColors.<token>` or the theme-aware `AppColorsTheme.of(context).<token>`.
- IMPORTANT: For **dark mode support**, always use `AppColorsTheme.of(context)` in widget `build()` methods:
  ```dart
  final c = AppColorsTheme.of(context);
  // Then use: c.white, c.chocolate, c.lightBlue, etc.
  ```
- Deprecated aliases exist (`burgundy`, `textPrimary`, `accentBlue`, etc.) — do **not** use them in new code.

### Spacing — `AppSpacing`

```dart
AppSpacing.xs  //  4.0
AppSpacing.sm  //  8.0
AppSpacing.md  // 16.0
AppSpacing.lg  // 24.0
AppSpacing.xl  // 32.0
```

- Use `EdgeInsets` with spacing tokens: `EdgeInsets.all(AppSpacing.md)`
- Use `SizedBox(height: AppSpacing.sm)` for vertical gaps

### Border Radius — `AppRadii`

```dart
AppRadii.xs      //  4px — progress bars, small indicators
AppRadii.sm      //  8px — chips, small rounded containers
AppRadii.small   // 12px — cards, input containers, badges
AppRadii.medium  // 16px — section cards, settings cards
AppRadii.large   // 20px — large containers, tab selectors
AppRadii.full    // 100px — circular elements (avatars, round buttons)
AppRadii.pill    // 999px — fully rounded (buttons, inputs, toggles)
```

- IMPORTANT: **Never use `BorderRadius.circular(N)` with a raw number.** Always use `const BorderRadius.all(AppRadii.<token>)`.
- For partial radii: `const BorderRadius.vertical(top: AppRadii.medium)`

### Typography — `AppTextStyles`

```dart
AppTextStyles.heading1   // 28px, w600, chocolate
AppTextStyles.heading2   // 24px, w600, chocolate
AppTextStyles.heading3   // 18px, w600, chocolate
AppTextStyles.body       // 14px, w400, chocolate
AppTextStyles.bodyMuted  // 14px, w400, chocolate (apply reduced opacity)
AppTextStyles.caption    // 12px, w400, chocolate
AppTextStyles.badge      // 12px, w600, white
AppTextStyles.button     // 16px, w600, white
```

- Override color for dark mode: `AppTextStyles.heading1.copyWith(color: c.chocolate)`
- For muted text: `.copyWith(color: c.chocolate.withValues(alpha: 0.5))`

### Shadows — `AppShadows` (neumorphic)

```dart
AppShadows.neumorphicOuter  // raised card effect
AppShadows.neumorphicInner  // inset/pressed effect
```

- Apply via `BoxDecoration(boxShadow: AppShadows.neumorphicOuter)`

---

## Component Patterns

### Widget Architecture

- IMPORTANT: Prefer `StatelessWidget` unless local mutable state is required
- IMPORTANT: All widgets must use `const` constructors with `super.key`
- Use `final` for all widget fields
- Mark required props with `required`; optional props use nullable types (`Type?`)

```dart
class MyWidget extends StatelessWidget {
  const MyWidget({
    super.key,
    required this.label,
    this.onTap,
  });

  final String label;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) {
    final c = AppColorsTheme.of(context);
    // ...
  }
}
```

### Naming Conventions

- **Files:** `snake_case.dart` (e.g., `primary_button.dart`)
- **Classes:** `PascalCase` (e.g., `PrimaryButton`)
- **Private helpers:** Prefix with `_` (e.g., `_NavItem`)
- **Properties:** `camelCase` (e.g., `onPressed`, `backgroundColor`)

### Composition

- Prefer **composition over inheritance** — compose widgets from smaller widgets
- Use private helper widgets (prefixed with `_`) for internal sub-components
- One public widget per file; private helpers may coexist in the same file

### Import Conventions

- Use package imports: `import 'package:pet_circle/widgets/primary_button.dart';`
- No barrel files — import each widget directly
- Group imports: Flutter SDK, third-party packages, project imports

---

## Styling Approach

### Button System

`PrimaryButton` supports two variants via `PrimaryButtonVariant`:

```dart
// Filled (default): chocolate bg, white text
PrimaryButton(label: 'Sign up', onPressed: () {})

// Outlined: white bg, chocolate text, subtle border
PrimaryButton(label: 'Cancel', variant: PrimaryButtonVariant.outlined, onPressed: () {})

// Smaller action buttons: adjust minHeight and borderRadius
PrimaryButton(label: 'Measure', minHeight: 42, borderRadius: 100, onPressed: () {})
```

- IMPORTANT: Always use `PrimaryButton` for buttons — do not create custom `ElevatedButton` instances inline
- Use `variant: PrimaryButtonVariant.filled` for primary actions, `.outlined` for secondary

### Dropdown Pattern

Use `AppDropdown` for labeled dropdown selectors (e.g., breed, role, diagnosis):

```dart
AppDropdown(
  label: 'Breed',
  value: selectedBreed,
  onTap: _toggleDropdown,
  isOpen: _isOpen,
  chevronController: _chevronController,
)
```

### Settings Row Pattern

Use `SettingsRow` for settings-style list items with icon + title + trailing action:

```dart
SettingsRow(
  iconAsset: 'assets/figma/settings_moon.svg',
  title: 'Dark mode',
  description: 'Toggle dark theme',
  trailing: TogglePill(isOn: isDark),
  onTap: toggleDarkMode,
)
```

### Neumorphic Design System

This project uses a custom **neumorphic** design language. Key characteristics:
- Soft raised/inset shadows (`AppShadows.neumorphicOuter` / `neumorphicInner`)
- Rounded corners (`AppRadii.medium` default for cards)
- Warm color palette (off-white backgrounds, chocolate text)
- Pill-shaped inputs and buttons (`AppRadii.pill`)

### Applying Styles

```dart
Container(
  padding: const EdgeInsets.all(AppSpacing.md),
  decoration: BoxDecoration(
    color: c.white,
    borderRadius: const BorderRadius.all(AppRadii.medium),
    boxShadow: AppShadows.neumorphicOuter,
  ),
  child: Text('Hello', style: AppTextStyles.body),
)
```

### Global Theme

- Light theme: `buildAppTheme()` in `app_theme.dart`
- Dark theme: `buildDarkTheme()` in `app_theme.dart`
- Font: Google Fonts **Inter** (via `google_fonts` package)
- Input fields: Filled, pill-shaped, no visible border

### Responsive Patterns

- Use `LayoutBuilder` to calculate responsive grid columns based on available width
- Use `MediaQuery.of(context)` for keyboard-aware layouts
- No predefined breakpoints; responsive behavior is dynamic

---

## Asset Handling

### Asset Storage

- SVG and image assets: `assets/figma/` directory
- Asset path constants: `lib/theme/app_assets.dart` (`AppAssets` class)
- Asset declaration: `pubspec.yaml` under `flutter.assets`

### Using Assets

```dart
// SVG
SvgPicture.asset(AppAssets.welcomeGraphic, width: 248, height: 248);

// Raster image
Image.asset(AppAssets.petPlaceholder, width: 64, height: 64, fit: BoxFit.cover);

// Wrapped with error handling
AppImage(assetPath: 'assets/figma/pet.png', width: 100, height: 100);
```

### Figma MCP Asset Rules

- IMPORTANT: If the Figma MCP server returns a localhost source for an image or SVG, **use that source directly**
- IMPORTANT: **DO NOT** import/add new icon packages — all assets should come from the Figma payload or existing `assets/figma/` directory
- IMPORTANT: **DO NOT** use or create placeholders if a localhost source is provided
- Store downloaded assets in `assets/figma/`
- Add a constant in `AppAssets` for every new asset
- Register new asset paths in `pubspec.yaml` if adding new asset directories

### Icon System

Two icon systems are in use:

1. **SVG icons** (custom): Stored in `assets/figma/`, rendered with `flutter_svg`
   - Navigation: `nav_home.svg`, `nav_heartbeat.svg`, `nav_heart.svg`, `nav_message.svg`
   - Settings: `settings_chevron.svg`, `settings_configure.svg`, etc.

2. **Material Icons** (built-in): Referenced via `Icons.<name>` or string constants in `AppAssets`
   - Usage: `Icon(Icons.favorite_border, color: c.cherry)`

- Prefer SVG icons from Figma for custom graphics; use Material Icons only for standard UI affordances

---

## Localization

- Localization files: `lib/l10n/` (ARB format)
- Supported locales: English (`en`), Hebrew (`he`)
- Access strings via `AppLocalizations.of(context)!.<key>`
- IMPORTANT: All user-facing strings should use localization — do not hardcode display text

---

## Project-Specific Conventions

### State Management

- Global `ChangeNotifier` stores for shared state (7 stores in `lib/stores/`)
- `ValueNotifier` for global preferences (locale, dark mode)
- `ListenableBuilder` for reactive UI rebuilds
- See `state-management.mdc` for full patterns and store registry

### Authentication

- Firebase Auth is configured but **currently disabled** (`kEnableFirebase = false`)
- Auth flows exist in `lib/screens/auth/` and `lib/providers/auth_provider.dart`

### Routing

- Route constants defined in `lib/app_routes.dart`
- Navigation via `MaterialPageRoute` and `Navigator`

### Charting

- Uses `syncfusion_flutter_charts` for data visualization

### Dark Mode

- Fully supported via `AppColorsTheme` extension
- IMPORTANT: Always use `AppColorsTheme.of(context)` for colors in widgets to ensure dark mode compatibility
- Text styles may need `.copyWith(color: c.chocolate)` override for dark mode

---
> Source: [HiLuLiT/pet-circle](https://github.com/HiLuLiT/pet-circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
