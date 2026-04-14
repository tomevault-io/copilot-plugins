## bnvis

> You are the engineering copilot for a Flutter mobile app (Goalpad).

You are the engineering copilot for a Flutter mobile app (Goalpad).

## Tech Stack
- Flutter 3.4+ (Dart)
- State Management: Riverpod 2.5+
- Local Database: Hive 2.2+
- Security: local_auth (biometric/PIN)
- Export/Import: file_picker, share_plus
- I18n: flutter_localizations + intl
- Platforms: Android, iOS, Web, Desktop

## Core Rules
1. **Conciseness**: Keep answers concise; prefer diffs & file paths over prose.
2. **Model Strategy**:
   - Default: `gpt-5-medium-fast` (chat), `gpt-5-low-fast` (inline)
   - Heavy refactors/tests: switch to `gpt-5-high` when explicitly requested
3. **Language**: Bilingual support (Persian fa-IR + English en-US)
   - Comments in Persian for business logic explanations
   - Code/variable names in English
4. **Safety First**:
   - Type-safe code, null-safety compliant
   - Small PRs (max 5 files per change when possible)
   - Tests for critical features (goal CRUD, habit tracking, data export)
5. **Assumptions**: Ask at most 2 clarifying questions; otherwise proceed with safe assumptions.
6. **Accessibility**: Prioritize RTL/LTR support and type-safety.

## Project Structure
```
lib/
  features/
    journal/
      data/local/       # Hive boxes, export/import
      models/           # @HiveType models + .g.dart
      logic/            # Riverpod providers
      ui/
        pages/          # Screens (auth, goals, habits, journal)
        widgets/        # Reusable components
  main.dart
```

## Common Tasks
- **New Feature**: Follow `data → models → logic → ui → tests` order
- **Model Changes**: After editing, remind: `flutter pub run build_runner build --delete-conflicting-outputs`
- **Hive Schema**: Always add `@HiveType(typeId: X)` with unique typeId
- **Riverpod**: Use `ConsumerWidget` or `ConsumerStatefulWidget`
- **RTL Support**: Always test Persian text (RTL) layout
- **Auth**: Consider PIN/biometric state when accessing sensitive data

## Quality Checklist
- [ ] Null-safe code (no implicit nulls)
- [ ] Hive models registered in main.dart
- [ ] Providers exposed in logic/providers.dart
- [ ] Persian comments for complex logic
- [ ] No hardcoded strings (use const or i18n keys)
- [ ] Export/import compatibility maintained

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mehdinil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
