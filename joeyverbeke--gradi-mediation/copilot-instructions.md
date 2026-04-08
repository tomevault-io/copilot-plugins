## gradi-mediation

> - `firmware/esp32s3_audio_min/` implements the ESP32-S3 bridge; it now emits binary `AUD0` frames and obeys `PAUSE` / `RESUME` commands.

# Repository Guidelines

## Project Structure & Module Organization
- `firmware/esp32s3_audio_min/` implements the ESP32-S3 bridge; it now emits binary `AUD0` frames and obeys `PAUSE` / `RESUME` commands.
- Desktop modules live under `desktop_vad/`, `asr/` (whisper.cpp, Faster-Whisper, Vosk), `llm/`, and `tts/`, with orchestration in `controller/`.
- `scripts/` exposes validation CLIs (`esp_audio_tester.py`, `vad_test.py`, `asr_transcribe.py`, `session_controller.py`, etc.).
- Specifications, validation notes, and roadmap checkpoints are in `docs/`; runtime traces go to `logs/sessions/`.
- External sources belong in `third_party/` (see its README for clone/build instructions).

## Build, Test, and Development Commands
- `uv venv --python 3.10 --seed`  
  `uv pip install pyserial webrtcvad requests soundfile faster-whisper vosk` — baseline environment.
- `uv run scripts/esp_audio_tester.py --port /dev/gradi-esp-mediate record --seconds 5 --output esp_mic_test.wav` — exercises the binary mic stream (handles READY/PAUSE/RESUME automatically).
- `uv run scripts/asr_transcribe.py --asr-engine faster_whisper --fw-model-dir third_party/faster-whisper/models phrase01.wav` — smoke test Faster-Whisper; swap `--asr-engine` for `whisper_cpp` or `vosk` as needed.
- `uv run scripts/session_controller.py --port /dev/gradi-esp-mediate --kokoro-voice af_bella --max-cycles 1` — full Idle→Playback cycle; add `--verbose-esp` during debugging.

## Coding Style & Naming Conventions
- Python: 4-space indent, type hints, `snake_case` modules, `CamelCase` dataclasses, minimal but meaningful comments.
- Firmware: keep constants in `constexpr`, avoid heap allocations in realtime paths, and restrict UART output to the defined text commands.
- Log lines consumed by automation must remain single-line JSON, emitted via the controller.

## Testing Guidelines
- Manual validation is expected: capture WAV artifacts, transcript files, or session JSONL before merging.
- Future automated tests should live under `tests/`, mirror package paths, and use `pytest` with `test_<module>.py` naming.
- When diagnosing audio regressions, include the relevant `logs/sessions/session_*.jsonl` and note ASR engine / firmware build used.

## Commit & Pull Request Guidelines
- Use short, imperative commit titles (e.g., `Switch ESP stream to AUD0 frames`). Keep firmware, desktop, and docs changes separated when practical.
- Reference roadmap IDs (B1, C2, E1, etc.) in PR descriptions and list the validation commands you executed.
- PRs should summarize observed behavior, link to artifacts (WAV, JSONL), and call out remaining risks or follow-up tasks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joeyverbeke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joeyverbeke)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
