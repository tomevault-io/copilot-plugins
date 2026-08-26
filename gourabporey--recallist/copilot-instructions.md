## recallist

> > Full project context: [AGENTS.md](../AGENTS.md)

# Recallist — Copilot Instructions

> Full project context: [AGENTS.md](../AGENTS.md)

This is a Flutter spaced-repetition app. Use Material Design 3 patterns. The state management approach is `StatefulWidget` + `setState` — do not introduce BLoC, Provider, or Riverpod unless asked.

## Code Style

- Files: `snake_case.dart` | Classes: `PascalCase` | Methods/vars: `camelCase` | Private: `_camelCase`
- Boolean state: `_isLoading`, `_isSaving` (prefix `_is` / `_has`)
- Form controllers: suffix `Controller` (`_titleController`, `_notesController`)
- Prefer `const` constructors for stateless widgets
- Break large widgets into small single-purpose widgets in a `widgets/` sub-folder alongside the parent screen

## Feature Patterns

When adding a new feature:

1. Create a folder under `lib/features/<feature-name>/`
2. Complex sub-widgets go in a nested `widgets/` sub-folder
3. Register any new service in `lib/core/service_locator.dart` as a lazy singleton
4. Access services via `sl<ServiceType>()` (the `GetIt` instance named `sl`)

## Hive Models

When adding a new Hive model:

1. Annotate with `@HiveType(typeId: N)` — pick the next unused integer
2. Annotate each field with `@HiveField(N)` — never reuse or skip integers
3. Add `part '<filename>.g.dart';` at the top of the file
4. Run `dart run build_runner build --delete-conflicting-outputs`
5. Register the generated adapter in `HiveDataSource.init()`
6. Commit the generated `*.g.dart` file

## Repository Pattern

- Define an abstract interface in `lib/core/data/repositories/`
- Implement it with a `Local` prefix (e.g., `LocalItemRepository`)
- Wire the interface to the implementation in `service_locator.dart`
- Inject via `sl<AbstractInterface>()`, never the concrete class directly

## Navigation

Use `Navigator.push(context, MaterialPageRoute(...))` for screen transitions. Pass callbacks (`onItemUpdated`, `onItemDeleted`) as constructor parameters for child-to-parent communication. No GoRouter — keep it simple unless routing complexity grows.

## Notifications

After any item create/update/delete, call `sl<NotificationService>().rescheduleNotifications()`. Don't call `scheduleAllNotifications()` directly from UI code.

---
> Source: [gourabporey/recallist](https://github.com/gourabporey/recallist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
