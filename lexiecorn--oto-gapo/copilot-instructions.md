## oto-gapo

> Conventions for auto_route configuration and generation


## Routing with auto_route

- **Router definition**: `lib/app/routes/app_router.dart` uses `@AutoRouterConfig()` with routes list and extends `$AppRouter` from the generated file `app_router.gr.dart`.
- **Pages**: Annotate pages with `@RoutePage(name: 'SomePageRouter')` to match generated route types used in the router.
- **MaterialApp**: `AppView` builds `MaterialApp.router` and uses `getIt<AppRouter>().config()` to supply the router.
- **Navigation**: Use context extensions from auto_route (e.g., `context.router.push(...)`) with generated route classes like `HomePageRouter()`.
- **Generation**:
  - Ensure dev dependency `auto_route_generator` and `build_runner` exist (present in `pubspec.yaml`).
  - Generate routes with:
    ```bash
    dart run build_runner build --delete-conflicting-outputs
    ```
  - If you add or rename pages, re-run generation.

Tips:

- Keep route paths in `app_router.dart` as single source of truth.
- Use explicit names for routes to stabilize generated API across refactors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lexiecorn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lexiecorn)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
