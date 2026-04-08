## sauro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sauro is a macOS menu bar app (Swift + SwiftUI) that automatically detects calendar events from screen content. It captures screenshots every 5 seconds, performs OCR on them, sends the extracted text to a local LLM (via Ollama) to identify potential calendar events, and adds detected events to Apple Calendar. A macOS notification with an "Undo" button lets users revert false positives.

## Tech Stack

- **Language:** Swift 6
- **UI:** SwiftUI (macOS 14+ / Sonnet)
- **Build:** Xcode project / Swift Package Manager
- **OCR:** Apple Vision framework (VNRecognizeTextRequest)
- **Screenshots:** ScreenCaptureKit
- **Calendar:** EventKit (Apple Calendar integration)
- **Notifications:** UserNotifications framework
- **LLM:** Ollama (local model, accessed via HTTP API at localhost:11434)

## Build & Run

```bash
# Build from command line
xcodebuild -scheme Sauro -configuration Debug build

# Run tests
xcodebuild -scheme Sauro test

# Run a single test
xcodebuild -scheme Sauro -only-testing:SauroTests/TestClassName/testMethodName test
```

## Architecture

The app follows a pipeline architecture with these stages:

1. **ScreenCapture** — Uses ScreenCaptureKit to capture the full screen every 5 seconds
2. **OCREngine** — Feeds the captured image into Vision framework for text recognition
3. **EventDetector** — Sends OCR text to Ollama and parses the LLM response for structured event data (title, date/time, location, etc.)
4. **CalendarManager** — Uses EventKit to create events in Apple Calendar
5. **NotificationManager** — Posts macOS notifications with an "Undo Add" action; handles the undo callback to delete the event

The app runs as a **menu bar app** (no dock icon). The menu bar popover shows status, recent detections, and settings (Ollama model selection, capture interval, target calendar).

## Required macOS Permissions

The app needs these entitlements/permissions at runtime:
- **Screen Recording** (for ScreenCaptureKit)
- **Calendar Access** (for EventKit)
- **Notifications** (for UserNotifications)

## Key Conventions

- Use `async/await` and Swift concurrency throughout (no Combine or callback-based async)
- Keep LLM interaction behind a protocol so the provider can be swapped later
- Dedup detected events — don't re-add an event that was already added or dismissed by the user
- The Ollama prompt should request structured JSON output for reliable parsing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mkvoya)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mkvoya)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
