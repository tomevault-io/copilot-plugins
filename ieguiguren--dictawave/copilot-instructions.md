## dictawave

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan at
specs/001-double-tap-dictation/plan.md
<!-- SPECKIT END -->

(That plan and its siblings `research.md`, `data-model.md`, `contracts/` in the same directory
hold the config schema, systemd unit contract, and installer CLI contract in detail.)

## What this is

DictaWave is a system-wide double-tap voice dictation daemon for Linux (X11 only, Wayland out
of scope). Double-tap a configured key anywhere → record → transcribe with Whisper → inject text
at the cursor via xdotool. See `README.md` for the full user-facing picture (install, usage,
troubleshooting).

## Commands

```bash
pip install --user --break-system-packages pytest ruff   # dev deps (openai-whisper/torch/PyGObject assumed present)
python3 -m pytest tests/ -q                               # all tests
python3 -m pytest tests/unit/test_stt.py -q                # single file
python3 -m pytest tests/unit/test_stt.py::test_name -q     # single test
ruff check dictawave/                                       # lint (line-length 100, py312, rules E/F/I/W/UP)
python3 scripts/detect_key.py [--write]                    # print/save evdev key name+code for the trigger
dictawave --reconfigure                                      # rerun key detection, prompt to restart daemon
systemctl --user status dictawave                           # daemon is a systemd user service
journalctl --user -t dictawave -f                            # live daemon logs
```

`tests/integration/test_cloud_stt_live.py` is a *real* (non-mocked) end-to-end test: it spins up
a genuine `http.server` on 127.0.0.1 and parses the multipart request with the stdlib, so it
catches wire-format bugs a mocked `urlopen` cannot. No credentials or network needed — it runs in
the normal test loop; `tests/integration/test_session_flow.py` mocks the pipeline instead.

## Architecture

Single-process daemon, one background worker thread per dictation session, driven by a small
state machine in `dictawave/daemon.py::Daemon`. Everything else is a narrow adapter around one
OS/library boundary:

```
dictawave/hotkey.py    — evdev global key listener + double-tap timing (DoubleTapDetector)
dictawave/audio.py     — sounddevice capture (16kHz mono) → AudioChunk queue
dictawave/vad.py       — webrtcvad silence detection (SilenceDetector), 3.5s timeout
dictawave/stt.py       — WhisperEngine (local, CUDA) and CloudWhisperEngine (HTTP fallback)
dictawave/injection.py — xdotool text injection + pyatspi focused-widget/password-field checks
dictawave/hud.py       — optional GTK4 floating volume overlay (degrades to no-op if GTK4 absent)
dictawave/clipboard.py — xclip fallback when no text field is focused
dictawave/notify.py    — notify-send wrapper
dictawave/config.py    — Config dataclass, tomllib load / tomli-w write, validate()
dictawave/types.py     — shared dataclasses/enums (DaemonState, DictationSession, AudioChunk, ...)
```

Control flow: `HotkeyListener` fires `_on_key_down` on every keypress; `Daemon` tracks
`DaemonState` and only starts a session on a genuine double-tap. `_begin_session` spins up
`AudioCapture` plus a `_session_worker` thread that pulls `AudioChunk`s off a queue, runs VAD to
segment phrases, calls `flush_phrase` to transcribe each phrase (`stt.py`) and deliver text
incrementally via `_deliver_text` → `injection.py`. A session ends on a second double-tap or on
silence timeout; `_end_session`/`_finalize_session` tear down capture and reset state to IDLE.

**STT backend selection**: `stt.py` exposes two interchangeable engines — `WhisperEngine`
(local, lazy-loaded onto CUDA on first use, unloaded after `idle_unload_seconds` of inactivity)
and `CloudWhisperEngine` (posts WAV to any OpenAI-compatible `/v1/audio/transcriptions`
endpoint, config under `[cloud_fallback]` in `config.toml`). The daemon falls back to cloud
automatically whenever the local model fails to load or transcribe — see README's "Using a
remote Whisper instead of local" section for the exact fallback semantics.

**Password-field safety**: `injection.py::inspect_focus` walks the AT-SPI tree before injecting;
dictation is refused into password fields. Some apps (Electron, terminals) expose no AT-SPI
tree, so this guard is best-effort, not a hard boundary — don't rely on it as the sole safety
mechanism for a feature.

**Config**: single TOML file at `~/.config/dictawave/config.toml`, loaded via `config.load_config`
and validated in `Config.validate()`. Schema is documented in
`specs/001-double-tap-dictation/contracts/config-schema.md`, not duplicated in code comments.

## Constitution / constraints (from the spec)

These are enforced requirements from `specs/001-double-tap-dictation/plan.md`'s Constitution
Check, not just style preferences — changes touching audio handling or the trigger key should
keep them true:

- No hardcoded trigger key — always user-configured at install (`install.sh`) or via
  `scripts/detect_key.py --write`.
- No background audio recording; raw audio is discarded immediately after each transcription
  call (never persisted to disk).
- STT/network failure must degrade gracefully (notify + pass-through), never crash the daemon.
- Cloud fallback endpoints must use TLS — plain HTTP is rejected at config validation time.

---
> Source: [ieguiguren/dictawave](https://github.com/ieguiguren/dictawave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
