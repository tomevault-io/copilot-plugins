## 04-ui-components

> When creating a new component, or trying to understand how components are structured

# UI Components

The application uses Flutter's Material Design components and custom widgets.

## Component Organization
- Feature-specific components are stored in the `components` directory of each feature
- [lib/shared/components/](mdc:lib/shared/components) contains reusable components used across the application

## Screens
- Screens represent full pages in the application
- Each feature typically has one or more screens
- Screen files are named with a `_screen.dart` suffix
- Examples:
  - [lib/features/track/track_screen.dart](mdc:lib/features/track/track_screen.dart)
  - [lib/features/track/new_routine_screen.dart](mdc:lib/features/track/new_routine_screen.dart)

## Theming
- [lib/theme.dart](mdc:lib/theme.dart) contains the application's theme configuration
- Use the theme's colors, text styles, and other design tokens for consistency

## Design Patterns
- Widgets should be composable and reusable
- Maximum one widget per file
- Extract common UI patterns into shared components
- Follow Flutter's widget hierarchy guidelines for performance

---
> Source: [shareefhadid/traak](https://github.com/shareefhadid/traak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
