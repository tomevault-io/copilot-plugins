## 01-project-overview

> Understanding the project structure and identifying entrypoints

# Traak - Project Overview

This is a Flutter application that helps sprinters track workout routines and get insights to help them improve their performance.

## Main Entry Points
- [lib/main.dart](mdc:lib/main.dart) - Application entry point that initializes services and launches the app
- [lib/app.dart](mdc:lib/app.dart) - Root app widget that sets up theming and routing
- [lib/router.dart](mdc:lib/router.dart) - Routing configuration using go_router
- [lib/theme.dart](mdc:lib/theme.dart) - Theme configuration for the app

## Project Structure
The codebase follows a feature-first organization:
- `lib/features/` - Contains feature modules with their screens, models, and logic
- `lib/shared/` - Shared logic used across features
- `lib/constants/` - Constants and configuration values

---
> Source: [shareefhadid/traak](https://github.com/shareefhadid/traak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
