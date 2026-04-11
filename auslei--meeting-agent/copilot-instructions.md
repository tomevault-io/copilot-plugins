## meeting-agent

> Automate the lifecycle of joining a WeMeet meeting from a WeChat message and recording the audio to an MP3 file.

# GEMINI.md - WeMeet Automation Project

## Project Goal
Automate the lifecycle of joining a WeMeet meeting from a WeChat message and recording the audio to an MP3 file.

## Core Mandates & Implementation Status

### 1. Robustness & Error Recovery
- **Implementation:** Status transitions are verified using OCR in `src/wemeet/joiner.py`.
- **Strategy:** Every critical action (find window, join, verify) includes retries and detailed logging via `src/common/logger.py`.
- **Validation:** `verify_in_meeting` method in `WeMeetJoiner` checks for active meeting controls (e.g., "Mute", "Leave") to confirm state.

### 2. Cross-Platform & Resolution Independence
- **Implementation:** 
    - **Window Management:** Uses `pywinctl` for cross-platform window localization.
    - **UI Interaction:** Uses URL schemes (`wemeet://`) as the primary join method to bypass resolution issues.
    - **Fallback:** OCR-based interaction in `src/common/gui_interactor.py` avoids hardcoded pixel coordinates.
- **Support:** Windows, macOS, and Linux (tested for window detection and OCR).

### 3. Logging & Monitoring
- **Implementation:** Centralized logging using `loguru` in `src/common/logger.py`.
- **Output:** Synchronized console and file logging (`cu_agent.log`) with tracebacks for debugging.

### 4. Audio Recording
- **Implementation:** `src/common/audio_recorder.py` uses `sounddevice` for loopback capture.
- **Independence:** Recording runs in a dedicated background thread to ensure UI lag doesn't affect audio quality.
- **Format:** Direct conversion to MP3 via `pydub` and `FFmpeg`.

## Technical Architecture (Modular)
- **`src/common/`**: Shared utilities (Logger, GUI Interactor, Audio Recorder).
- **`src/wechat/`**: WeChat-specific monitoring logic.
- **`src/wemeet/`**: WeMeet-specific joining and verification logic.
- **`src/tests/`**: Environment validation and unit tests.

## Development Standards
- **Standardized Codebase:** Adheres to `GLOBAL_STANDARDS.md` (type hints, modularity, `uv` for management).
- **Validation:** Always run `src/tests/test_env.py` before deploying in a new environment to verify Tesseract and window visibility.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/auslei)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/auslei)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
