## retrowaves

> This document provides a comprehensive guide for developers working on the Retrowaves project. It outlines the project's architecture, setup instructions, and development conventions.

# Retrowaves Gemini Guide

This document provides a comprehensive guide for developers working on the Retrowaves project. It outlines the project's architecture, setup instructions, and development conventions.

## Project Overview

Retrowaves is a sophisticated, Python-based radio automation and broadcasting system. It is composed of two main, independent components:

*   **Station**: The "brain" of the operation. It's a DJ-driven, event-timed, intent-based radio automation engine. It handles music selection, scheduling, and decoding of audio files into a raw PCM audio stream. The Station operates on a "THINK/DO" cycle, where "THINK" involves planning the next audio segment, and "DO" involves executing the playout.

*   **Tower**: The "transmitter" of the operation. It's a 24/7, process-isolated, HTTP-based audio transmission service. It takes the PCM audio stream from the Station, encodes it into MP3 format using FFmpeg, and broadcasts it over HTTP. The Tower includes a fallback mechanism to ensure a continuous stream even if the Station is offline.

### Architecture and Data Flow

1.  The **Station** selects and decodes audio files (e.g., MP3s) into a PCM audio stream.
2.  The PCM stream is sent to the **Tower** via a Unix domain socket.
3.  The **Tower** receives the PCM stream and encodes it to MP3 using FFmpeg.
4.  The Tower broadcasts the MP3 stream to any connected HTTP clients (like OBS, VLC, etc.).

The system is designed to be robust and resilient, with the Station and Tower able to operate independently.

## Building and Running

### Dependencies

The project's dependencies are listed in `requirements.txt`. To install them, use pip:

```bash
pip install -r requirements.txt
```

The project also has a dependency on **FFmpeg** for audio encoding and decoding. Ensure that FFmpeg is installed and available in your system's PATH.

### Configuration

The Tower component uses a `.env` file for configuration. A sample file can be found at `tower/tower.env.example`. Copy this file to `tower/tower.env` and customize it as needed.

### Running the Services

The Station and Tower are designed to run as separate services.

*   **To run the Station:**

    ```bash
    python3 run_station_dev.py
    ```

*   **To run the Tower:**

    ```bash
    python3 run_tower_dev.py
    ```

There are also convenience scripts to start and stop the services:

*   `start.sh`: Starts both the Station and Tower.
*   `shutdown.sh`: Stops both services.

## Development Conventions

### Code Style

The project follows standard Python coding conventions.

### Testing

The project uses `pytest` for testing. The `pytest.ini` file contains the configuration for `pytest`. To run the tests, execute the following command:

```bash
pytest
```

There are also scripts related to contract testing, such as `execute_contract_tests.py` and `run_contract_tests.py`.

### Documentation

The project has extensive architecture documentation in Markdown files, such as:

*   `ARCHITECTURE_DIAGRAM.md`
*   `ARCHITECTURE_STATION.md`
*   `ARCHITECTURE_TOWER.md`

These documents are the canonical source of truth for the system's architecture and should be consulted for in-depth understanding.

---

## Session History

### Session 2025-12-27
- Branch: main
- Accomplishments:
  - Standardized Tower documentation and tools on 1024-sample PCM frames (4096 bytes) at 48 kHz (≈21.333ms).
  - Replaced video-centric session closeout doc with engineering-oriented `SESSION_CLOSE.md`.
- Key Decisions:
  - Canonical PCM cadence is 1024-sample frames; Tower docs/tools must match the contract and code.
- Next Steps:
  - Audit remaining references to 1152/4608/24ms across repo (tests/docs).
  - Run contract/integration tests to validate end-to-end behavior.

---
> Source: [slbailey/retrowaves](https://github.com/slbailey/retrowaves) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
