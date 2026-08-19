## flutter-stt

> - This is a single Flutter application (Flutter 3.44.8 / Dart 3.12.2); the package SDK constraint is `^3.12.2`.

# AGENTS.md

## Project

- This is a single Flutter application (Flutter 3.44.8 / Dart 3.12.2); the package SDK constraint is `^3.12.2`.
- `lib/main.dart` is the sole app entrypoint and currently contains the full Material 3 speech-recognition UI and its state.
- Speech recognition uses `speech_to_text`; it prefers `zh_TW`, falls back to the device default locale, and retains at most three distinct transcripts in memory.

## Verification

- Run `flutter analyze` and `flutter test` from the repository root. Run only the focused widget test with `flutter test test/widget_test.dart`.

## Native Speech Support

- Android speech support is configured in `android/app/src/main/AndroidManifest.xml` with `RECORD_AUDIO`, `INTERNET`, and a `RecognitionService` query; preserve or update these when changing speech functionality.
- iOS `Runner/Info.plist` currently has no `NSMicrophoneUsageDescription` or `NSSpeechRecognitionUsageDescription`; add both before expecting speech recognition to work on iOS.

---
> Source: [howfan31/flutter_STT](https://github.com/howfan31/flutter_STT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
