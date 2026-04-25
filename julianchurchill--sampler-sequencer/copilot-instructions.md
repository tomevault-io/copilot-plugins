## sampler-sequencer

> **Never commit directly to `main`.** All changes must go through feature branches and be merged via pull request.

# Sampler-Sequencer Development Guidelines

## Branching Policy

**Never commit directly to `main`.** All changes must go through feature branches and be merged via pull request.

### Workflow

1. Create a feature branch from `main`:
   ```
   git checkout main
   git pull origin main
   git checkout -b feature/your-feature-name
   ```
2. Make your changes and commit to the feature branch.
3. Push the branch and open a pull request targeting `main`.
4. Merge only after the CI build passes (GitHub Actions runs `flutter build apk --release`).

### Branch naming

Use descriptive prefixes:
- `feature/` — new functionality
- `fix/` — bug fixes
- `chore/` — tooling, dependencies, config changes

## Versioning and Changelog

Every pull request that changes user-facing behaviour **must**:

1. **Update `CHANGELOG.md`** — add an entry under a new version heading (or an `[Unreleased]` section if the version hasn't been decided yet). Follow the [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format: `### Added`, `### Changed`, `### Fixed`, `### Removed`.

2. **Bump the version in `pubspec.yaml`** (`version: MAJOR.MINOR.PATCH+BUILD`) following [Semantic Versioning](https://semver.org/):
   - `PATCH` — bug fixes, no new features
   - `MINOR` — new backwards-compatible features
   - `MAJOR` — breaking changes
   - `BUILD` — increment by 1 on every release (Android `versionCode`)

Pure housekeeping changes (CI config, docs, tooling) do not require a version bump, but should still note the change in `CHANGELOG.md` if it affects developers.

## TODO

When completing a task from `TODO.md`, mark it `- [x]` and move it to the **bottom** of its section, but **above** any already-completed `[x]` items there, so the newest completion sits at the top of the completed group.

## Pre-Commit Code Review

Before committing any code change, perform a self-review of the diff. Work through the checklist below and fix any issues found before proceeding with the commit.

### Style

- Dart code follows the [Dart style guide](https://dart.dev/guides/language/effective-dart/style): `lowerCamelCase` for variables/methods, `UpperCamelCase` for types, `_private` prefix for private members.
- No commented-out code, debug prints, or TODO comments left in (unless the TODO is tracked and intentional).
- Consistent formatting — run `dart format` if in doubt.

### Design

- New logic is placed in the correct layer: business logic in `SequencerModel`, DSP/audio in `dsp_utils.dart` or `AudioEngine`, UI-only state in widgets.
- No unnecessary abstractions introduced for a single use case.
- No speculative future-proofing (feature flags, unused parameters, over-engineered inheritance).
- Dependencies flow inward: UI → model → audio; never the reverse.

### Potential Issues

- No new platform-dependent code introduced into otherwise testable units.
- No hard-coded magic numbers without a named constant.
- No state mutation that bypasses `notifyListeners()`.
- No `async` gaps that could leave the UI in a stale state (e.g. awaiting a fire-and-forget call and treating it as complete).
- SharedPreferences keys are unique and follow the existing `_kPrefs*` naming convention.
- No security issues: no command injection, no untrusted input used in file paths or shell commands.
- If any drum generator in `dsp_utils.dart` was modified, `_kPresetCacheVersion` in `audio_engine.dart` has been incremented so the on-device WAV cache is invalidated.

#### AudioEngine invariants (must survive any rewrite of `init()` or `_rebuildPlayer()`)

Every `AudioPlayer` created in `AudioEngine` — sequencer players and the preview player — **must** have these three properties set before use:

| Property | Value | Reason |
|---|---|---|
| `setReleaseMode` | `ReleaseMode.stop` | Prevents `soundPool.release()` on sample completion, which would free the shared SoundPool and silence all other tracks mid-play. |
| `setAudioContext` | `AudioContextAndroid(audioFocus: AndroidAudioFocus.none)` | audioplayers' `FocusManager` requests `AUDIOFOCUS_GAIN` on every `play()` for **all** player modes including lowLatency/SoundPool. When any track triggers it steals focus, sending `AUDIOFOCUS_LOSS` to the other tracks which then stop themselves. Disabling focus management lets all 4 tracks play fully independently. This bug has been reintroduced twice (PRs #30-area and #33). |
| `setPlayerMode` | `PlayerMode.lowLatency` (sequencer) or `PlayerMode.mediaPlayer` (preview/trim) | SoundPool gives ~1 ms trigger latency; mediaPlayer is required for `seek()`. |

If you are touching `init()` or `_rebuildPlayer()`, verify all three are set on every player before committing.

#### Ping-pong retrigger (do not collapse back to one player per track)

Each track uses `_kSlotsPerTrack = 6` SoundPool players. On every untrimmed trigger the engine advances `_nextSlot[track]` and uses that slot's player — stopping only its *previous* stream (from ≥2 triggers ago, well into amplitude decay). The stream from the most recent previous trigger is left to play out naturally. This is the only way to avoid the retrigger click without a hardware-level crossfade: collapsing to one player per track forces `stop()` at peak amplitude on every rapid retrigger, producing an audible crack. The secondary slot (S=1) always stays `lowLatency`; only the primary slot (S=0) switches to `mediaPlayer` when trim is active.

**Why 6 slots, not 4:** at 120 BPM (125 ms/step), 4 slots caused slot reuse at exactly 500 ms — the same as the Kick 808 sample duration. `stop()` arrived in a race with SoundPool's natural-completion cleanup at the fade-out boundary, occasionally producing a click on the 6th+ consecutive hit. 6 slots push reuse to 750 ms, 250 ms past every preset's end.

**Use `play(source)` in the fast trigger path — do NOT replace it with `resume()`.** Although `play(source)` calls `setSource()` internally on every trigger (which appends to audioplayers' `urlToPlayers` cache), replacing it with `stop()` + `setVolume()` + `resume()` silently breaks playback. `SoundPoolPlayer.resume()` on Android checks a `prepared` flag before calling `start()`; `stop()` resets that flag, so `resume()` errors with "NotPrepared" and no new stream is started — the sample simply does not play, and the retrigger click returns. `play(source)` re-establishes `prepared` via `setSource()` before calling `resume()`, making it the only reliable restart path for a stopped SoundPool player.

### Tests

- Every new behaviour has a corresponding test written first (TDD).
- All `expect()` calls have a `reason:` string explaining what is being checked and why.
- Loop assertions include the failing index and value in the `reason:` string.
- No tests were deleted or disabled without explicit justification.

If the review surfaces issues, fix them before committing. Do not commit code that fails the review.

## Test-Driven Development (TDD)

All new features and bug fixes must follow a TDD workflow:

1. **Write a failing test first** — before writing any implementation code, add a test that describes the expected behaviour and confirm it fails.
2. **Write the minimum code to make it pass** — implement only what is needed to turn the test green.
3. **Refactor** — clean up the implementation while keeping all tests passing.

### Test locations

| What | Where |
|---|---|
| Pure constants | `test/constants_test.dart` |
| `SequencerModel` logic | `test/sequencer_model_logic_test.dart` |
| DSP / WAV utilities | `test/audio_engine_dsp_test.dart` |
| New logical units | `test/<unit_name>_test.dart` |

### Running tests

```
flutter test
```

The pre-commit hook (install with `sh scripts/install-hooks.sh`) runs `flutter test` automatically before every commit and blocks if any test fails. CI also runs `flutter test` before building the APK.

### What to test

- All business logic in `SequencerModel` (BPM, steps, velocity, mute, trim, etc.)
- Pure functions in `lib/audio/dsp_utils.dart`
- Any new pure/injectable logic added elsewhere
- Bug fixes must include a regression test that would have caught the bug

### What not to test

- Platform-dependent audio playback (`AudioEngine` internals requiring a real audio stack)
- Widget rendering (UI layout tests add fragility without proportionate value at this stage)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julianchurchill) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
