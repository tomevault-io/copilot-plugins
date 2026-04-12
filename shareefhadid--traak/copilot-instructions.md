## traak

> Creating a new feature or understanding the structure of a feature

# Feature Architecture

Each feature in the application is organized with a consistent structure:

## Features Directory
Features are located in [lib/features/](mdc:lib/features) and follow this structure:

- `screens/` or `*_screen.dart` - UI components that represent full screens/pages
- `components/` - Reusable UI components specific to the feature
- `models/` - Data models for the feature
- `repositories/` - Data access layer for the feature
- `view_models/` - Contains business logic and state management

## Main Features
- [Track](mdc:lib/features/track/track_screen.dart) - Feature for tracking workout routines
- [Insights](mdc:lib/features/insights) - Feature for viewing analytics and insights

## Development Patterns
When creating new feature components:
1. Place the feature in its own directory under `lib/features/`
2. Follow the established architecture pattern
3. Use shared components from `lib/shared/` when possible
4. Register routes in [router.dart](mdc:lib/router.dart)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shareefhadid)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shareefhadid)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
