## pkg-panel

> `lib/main.dart` bootstraps the app and wires persistence into the controller. Core UI lives in `lib/src/app.dart`. Business logic and shell integration live under `lib/src/services/`, and shared domain models live in `lib/src/models/`. Widget and parser tests live in `test/`, currently in `test/widget_test.dart`. Static icons for package managers are stored in `assets/package_icons/`. Platform runners are checked in under `windows/`, `linux/`, `macos/`, and `web/`, but the current product scope is Windows-first.

# Repository Guidelines

## Project Structure & Module Organization
`lib/main.dart` bootstraps the app and wires persistence into the controller. Core UI lives in `lib/src/app.dart`. Business logic and shell integration live under `lib/src/services/`, and shared domain models live in `lib/src/models/`. Widget and parser tests live in `test/`, currently in `test/widget_test.dart`. Static icons for package managers are stored in `assets/package_icons/`. Platform runners are checked in under `windows/`, `linux/`, `macos/`, and `web/`, but the current product scope is Windows-first.

## Build, Test, and Development Commands
Run `flutter pub get` after pulling dependency changes. Use `flutter run -d windows` for the main local workflow and `flutter run -d chrome` only for quick web checks. Run `flutter analyze` before opening a PR; lint rules come from `flutter_lints` via `analysis_options.yaml`. Run `flutter test` for the full test suite. Use `dart format lib test` to normalize Dart formatting before review.

## Coding Style & Naming Conventions
Follow standard Dart style: 2-space indentation, trailing commas where they improve formatter output, and one public type per conceptual unit. Use `UpperCamelCase` for classes and enums, `lowerCamelCase` for members, and `snake_case.dart` for file names such as `package_panel_controller.dart`. Keep UI text and shell-command behavior explicit; this app mixes Chinese labels with English identifiers, so preserve that split instead of translating code symbols.

## Testing Guidelines
Use `flutter_test` for widget and unit-style coverage. Add widget tests for UI state changes and focused parser tests for package-manager output handling. Name tests by behavior, for example `testWidgets('renders seeded package data', ...)` or `test('winget parser skips localized header rows', ...)`. Prefer fake `ShellExecutor` implementations over invoking real package managers in tests.

## Commit & Pull Request Guidelines
There is no established git history in this repository yet, so use short imperative commit subjects such as `Add cached latest-version store`. Keep commits scoped to one change. PRs should describe user-visible behavior, list validation commands run, and attach screenshots or recordings for UI changes. Link the relevant issue when one exists and call out any Windows-only assumptions.

## Security & Configuration Tips
Do not hardcode machine-specific paths or package-manager state. Keep command execution inside `ShellExecutor`, and treat external command output as untrusted input that must be parsed defensively.

## Agent Workflow
Once you have completed all the tasks given by the user and summarized them, the last thing you need to do is to use the notification skill to notify the user that the task is complete

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/caolib)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/caolib)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
