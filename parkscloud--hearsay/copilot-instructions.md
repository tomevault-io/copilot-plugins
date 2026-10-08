## hearsay

> Project guide for AI-assisted development sessions. This file is the durable, portable project memory — update it when workflows or architecture change. **This repo is public: never put personal information, machine-specific paths, or incident details in this file or any committed file.**

# CLAUDE.md

Project guide for AI-assisted development sessions. This file is the durable, portable project memory — update it when workflows or architecture change. **This repo is public: never put personal information, machine-specific paths, or incident details in this file or any committed file.**

Hearsay is a Windows tray app: WASAPI loopback + microphone capture → faster-whisper (fully local) → source-labeled markdown transcripts.

## Build & release

**Build the installer after every code change.** From the project root (~2 minutes total):

1. `pyinstaller --noconfirm Hearsay.spec` → `dist\Hearsay\` (`build.bat` wraps this; the spec is the canonical build definition)
2. `"C:\Program Files (x86)\Inno Setup 6\ISCC.exe" installer.iss` → `installer_output\HearsaySetup.exe`

The version lives in **three** files that must stay in sync: `src/hearsay/__init__.py` (`__version__`), `src/hearsay/constants.py` (`APP_VERSION`), `installer.iss` (`AppVersion`). GitHub release flow is in `RELEASING.md`.

`gh release create` makes the version tag on GitHub only — run `git fetch --tags` before relying on local tags (they are usually behind).

The installer pins a permanent `AppId` GUID in `installer.iss` — **never change or regenerate it**, or Windows stops recognizing installed copies as the same app across upgrades. Introduced in v1.1.2; installs from v1.1.1 and earlier (no explicit `AppId`, so a hash-derived identity) don't upgrade across that boundary and must be uninstalled from Add/Remove Programs first — a one-time step that release notes should call out.

Run from source: `python -m hearsay` with `src` on `PYTHONPATH`.

## Conventions

- Bug and feature tracking uses GitHub Issues (`gh issue ...`); release notes reference them ("closes #N"). Scratch incident write-ups (`ISSUE_*.md` at the repo root) are local working notes — gitignored, leave them untracked. Durable resolutions belong in GitHub issues, release notes, and commit messages.
- Stage files explicitly when committing; avoid `git add -A`.
- Keep documentation consistent: when changing the app or cutting a release, fix any drift across `README.md`, `RELEASING.md`, and this file in the same pass — stale version examples, build/release steps, and the `src/` file tree (add new modules, drop removed ones).
- Verify changes with `python tests/test_pipeline_writer.py` (fast; stub engine, no hardware). Before releasing anything that touches audio, also run `python scripts/manual_device_check.py` (~3 min; records real devices while playing TTS through the speakers).
- **The app may be recording a real meeting while you work.** Before launching test instances, running the device check, or playing TTS audio, confirm no session is active — check for recent `Chunk N transcribed` lines in `%APPDATA%\Hearsay\logs\hearsay_<date>.log` (that log is also the first stop when diagnosing a bad session). Kill test instances by PID only; `taskkill /IM Hearsay.exe` would hit the user's real instance.

## Architecture (the non-obvious parts)

- **Threading:** background threads subclass `StoppableThread` (`utils/threading_utils.py`); UI updates from threads go through `safe_after(root, ms, callback)`.
- **Audio flow:** `AudioRecorder` uses callback-driven capture (never blocking reads — stop must stay <1s) and cuts wall-clock ~30s windows from per-source buffers that keep a 1s overlap tail. It queues `AudioChunk(index, window_start, parts: {source → ndarray})`, system-first.
- **Device selection:** capture is WASAPI throughout. Mic-only uses sounddevice's WASAPI input; loopback and "Both" use PyAudioWPatch. Config stores device *names* (`mic_device_name`/`loopback_device_name`), not indices — the recorder resolves a name to a concrete device (and its native sample rate) at open time, tolerant of small library-to-library name differences, falling back to the system default. Empty name = default. Legacy MME devices are intentionally excluded (they can open but deliver nothing). The mic-only path opens **and starts** the stream inside a retry that re-initializes PortAudio (`sd._terminate()`/`_initialize()`) and re-resolves the device between attempts — so a stale cached device handle (a USB webcam that slept/re-seated → `-9999` "Unanticipated host error" on stream *start*) self-heals in seconds instead of needing an app restart.
- **Pipeline:** transcribes each source in a window separately (no mixing), applies per-source overlap dedup, then a fuzzy echo guard — a mic segment whose words ≥80% match the same window's system text (in order, `difflib`) is dropped as speaker echo. Emits one merged, source-tagged `TranscriptionResult` (`segments` carry `source`; `window_start` is wall-clock seconds).
- **Writer:** `MarkdownWriter` inserts `**Remote:**` / `**Local:**` labels on every source switch (single-source sessions naturally get one label at the top); within a source, a ≥2s gap starts a new paragraph. `post_process()` cleans text per label-block so labels survive the filler/duplicate/whitespace passes. Empty sessions write "No speech was captured during this session."
- **Session lifecycle:** fresh queues per session (no cross-session bleed); teardown waits for the recorder thread to actually exit before the next session may start; device opens retry 5× with backoff; recorder death surfaces via `on_fatal` → tray notification + session stop, backed by a 5s `is_alive()` watchdog.
- **Silent-capture watchdog:** a device can open, report "active," and deliver *no* audio (muted, unplugged, blocked, or a wedged USB webcam). `_SilenceMonitor` in the recorder tracks time since the last non-silent window and fires `on_no_audio` after ~60s (re-alerting every ~120s) → tray warning + live-view status, but keeps recording so the session recovers if the device returns. This is the "recording failures must be loud" invariant for the *alive-but-silent* case (distinct from `on_fatal`, which stops on a dead recorder).
- **Timestamps:** transcript speaker labels are untimestamped (v1.1.5+) — `[m:ss]` stamps surface only in the live view. Segment timing still drives the paragraph-gap logic: window starts are wall-clock accurate; offsets *within* a window are approximate when system audio is intermittent (captured audio is positioned at the window start). Known, accepted — not a bug.
- **Audio is never persisted to disk** (by design) — a failed transcription is unrecoverable, which is why recording failures must be loud. Opt-in WAV retention was considered and deliberately deferred; the design sketch lives in GitHub issue #11 (default-off setting, stereo WAV with system=left / mic=right).
- Config and logs live under `%APPDATA%\Hearsay\`; Whisper models are cached there too.

---
> Source: [parkscloud/Hearsay](https://github.com/parkscloud/Hearsay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
