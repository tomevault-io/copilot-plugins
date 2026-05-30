## 05-navigation

> Creating new routes or understanding navigation

# Navigation

The application uses the go_router package for navigation.

## Router Configuration
- [lib/router.dart](mdc:lib/router.dart) contains the router configuration
- Routes are defined in a hierarchical structure
- Each screen has a unique path

## Navigation Patterns
- Navigation should be performed using the router's methods:
  ```dart
  context.go('/path');
  context.push('/path');
  ```
- Nested routes should follow the parent-child relationship in the router configuration

## Route Parameters
- Route parameters can be passed using path parameters or query parameters
- Access route parameters in the screen widget using GoRouterState

## Adding New Routes
When adding a new screen:
1. Create the screen widget in the appropriate feature directory
2. Add the route in [lib/router.dart](mdc:lib/router.dart)
3. Use the existing pattern for parent-child relationships if necessary

---
> Source: [shareefhadid/traak](https://github.com/shareefhadid/traak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
