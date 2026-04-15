## msiamn-dev

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Flutter portfolio web app (also targeting iOS, Android, macOS, Windows, Linux) for software engineer Siam. Deployed to Firebase Hosting and an FTP server via GitHub Actions.

## Commands

### Setup
```bash
flutter pub get
dart run build_runner build                          # code generation (Freezed, JSON, Riverpod)
dart run easy_localization:generate -S assets/translations -f keys -O lib/src/localization/generated -o locale_keys.g.dart
dart run easy_localization:generate -S assets/translations -f json -O lib/src/localization/generated -o locale_json.g.dart
```

### Run & Build
```bash
flutter run -d chrome
flutter build web --web-renderer canvaskit --release --no-tree-shake-icons
```

### Test & Lint
```bash
flutter test
flutter test test/widget_test.dart                  # single test file
flutter analyze
flutter format lib/
```

### Deploy
```bash
firebase deploy --only hosting
```

## Architecture

**Feature-first clean architecture** under `lib/src/features/`. Each feature contains:
- `domain/` — Freezed immutable models with JSON serialization
- `presentation/` — Widgets split by breakpoint (`_desktop`, `_tablet`, `_mobile`)
- `provider/` (main feature only) — Riverpod providers

**Features:**
- `introduction/` — Hero section: name, bio, contact links, resume download
- `experience/` — Work history timeline
- `project/` — Projects showcase with screenshots and tech stack
- `about/` — Personal bio
- `main/` — App shell, routing, and global providers

**Global providers** (`lib/src/features/main/provider/`):
- `darkModeProvider` — Theme toggle persisted via SharedPreferences
- `brightnessControllerProvider` — System brightness detection
- `scrollControllerProvider` — Smooth scroll between sections
- `sectionKeyProvider` — Section anchor keys for navigation

**Responsive breakpoints** (`lib/src/common/widgets/responsive.dart`):
- Desktop ≥ 1024px, Tablet 640–1024px, Mobile < 640px

**Theming:** FlexColorScheme + Material 3. Colors and font sizes in `lib/src/constants/`. Font: Nunito (Google Fonts).

**Localization:** Single `assets/translations/en.json` source. Type-safe keys generated to `lib/src/localization/generated/locale_keys.g.dart`. Use `LocaleKeys.<key>.tr()` in widgets.

## Code Generation

Generated files (`*.g.dart`, `*.freezed.dart`) are excluded from linting. Re-run `build_runner` after modifying any Freezed model, JSON-serializable class, or Riverpod provider annotated with `@riverpod`.

All domain models use Freezed for immutability and equality. Riverpod providers use `@riverpod` annotations with auto-generated code.

## CI/CD

GitHub Actions (`.github/workflows/dart.yml`) runs on push/PR to `main`: generates code → builds web release → uploads artifacts → deploys via FTP using repository secrets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masnun-siam)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/masnun-siam)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
