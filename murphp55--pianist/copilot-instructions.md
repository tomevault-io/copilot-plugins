## pianist

> Flutter/Dart piano practice companion with MIDI input, metronome, sheet music rendering, and ear training drills.

# Pianist — Agent Notes

## One-liner
Flutter/Dart piano practice companion with MIDI input, metronome, sheet music rendering, and ear training drills.

## Run it
```
flutter pub get
flutter run -d windows        # or: flutter run -d android
flutter analyze               # used in place of tests; should pass clean
flutter build apk --release
```

## Where we are right now
- Last touched: 2026-04-19
- Working on: Experimenting with alternative sheet music renderers (Verovio, VexFlow) alongside the existing OSMD view — uncommitted new widgets in `lib/widgets/`, plus fetch scripts in `scripts/`.
- Known broken: Nothing known. `pubspec.yaml`, `windows/flutter/generated_plugin_registrant.cc`, and the OSMD bridge are modified in working tree.

*This section goes stale fast. Check `git log -5` and `git status` before trusting it.*

## Gotchas
- App is structured as **3 independent top-level tabs** (Daily Practice, Song Learning, Ear Training). Only Daily Practice uses `AppState`/Provider — the other two are self-contained. Don't assume central state.
- `NoteOnEvent` and `NoteFeedback` live in `services/practice_evaluator.dart`, NOT in `midi_service.dart`.
- Metronome timing tolerance is hardcoded ±20%. Wrong-timing penalty *un-counts* a correct note (decrements `expectedIndex` and `correctCount`) — can feel harsh.
- `MidiServiceFactory` silently falls back to `MockMidiService` if the platform plugin is missing. "MIDI not working" is the expected dev state; use `AppState.simulateNote()` to test.
- Metronome audio degrades silently to visual-only if `assets/sounds/metronome_click.mp3` is absent (file is not committed).
- Sheet music renders via OpenSheetMusicDisplay loaded from jsDelivr CDN inside a `flutter_inappwebview`, bridged through `assets/osmd/index.html`. Offline use requires bundling OSMD.
- Ear training audio uses a hidden WebView running Tone.js + Salamander samples from CDN; first Play click starts the audio context silently (browser autoplay policy).

## Non-obvious conventions
- All logging goes through `AppLogger` (services/app_logger.dart) — never `print`.
- All `AppState` mutations must call `notifyListeners()`; cancel `StreamSubscription`s in `dispose()`.
- `PlanFactory.buildDailyPlan(key, level)` is the single entry point for practice plans; levels 1–5 are cumulative. The old "Per-Key Extras" tab has been removed — level dropdown replaces it.

See README.md for project description, tech stack, and feature list.

---
> Source: [murphp55/Pianist](https://github.com/murphp55/Pianist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
