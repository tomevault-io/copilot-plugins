## htkit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HTKit is a Swift/SwiftUI library for real-time audio transcription on iOS and macOS using whisper.cpp. It provides Swift 6 strict concurrency compliance with thread-safe transcription access and live microphone streaming support.

**Platforms:** iOS 17.0+, iPadOS 17.0+, macOS 14.0+

## Build Commands

```bash
swift build              # Build the library
swift test --no-parallel # Run tests (requires test resources)
./format.sh              # Format all Swift code
```

## Test Setup

Test resources must be downloaded before running tests:

```bash
cd Tests/HTKitTests/Resources
wget https://files.bjnortier.com/HTKit/jfk.wav
wget https://files.bjnortier.com/HTKit/aragorn.wav
wget https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-tiny.bin
cd -
```

Tests use Swift Testing framework (not XCTest). Test accuracy is validated using Levenshtein distance with >80% similarity threshold.

## Architecture

### Concurrency Model

- **Actors:** `HTTranscriber` (whisper.cpp wrapper) and `HTStreamingAudioBuffer` (sample management)
- **@MainActor:** `HTJob`, `HTFileJob`, `HTStreamingJob` for UI observation
- **DispatchQueue with barriers:** `HTTranscription` and `HTAbortController` use this pattern instead of actors for C++ callback compatibility
- **@unchecked Sendable:** Used where C++ interop requires it

### Core Components

**HTTranscriber** - Actor wrapping whisper.cpp. Handles model loading, transcription execution, and C++ callbacks via `Unmanaged` pointers.

**HTJob** - Base `@Observable` class with state machine: `.created` → `.loadingModel` → `.transcribing` → `.stopping` → `.done`/`.error`

**HTFileJob** - Transcribes pre-loaded Float arrays (file samples)

**HTStreamingJob** - Real-time transcription using `HTStreamingEngine` protocol. Uses 29-second frames with 800-sample overlap and high-water mark tracking for frame boundary handling.

**HTTranscription** - Thread-safe accumulator for transcription segments. `appendAtHighWaterMark()` handles overlapping frames by updating existing content rather than duplicating.

**HTStreamingEngine** - Protocol for audio sources. `HTMicrophoneStreamingEngine` implements this using AVAudioEngine with format conversion to Whisper's 16kHz PCM requirement.

### Key Design Decisions

- Whisper operates on 30-second frames; streaming buffer manages overlaps automatically
- High-water mark tracking prevents duplicate transcription text at frame boundaries
- GPU acceleration via Metal with CPU fallback for simulator builds
- Thread pool leaves 2 efficiency cores free for system responsiveness

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bjnortier) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
