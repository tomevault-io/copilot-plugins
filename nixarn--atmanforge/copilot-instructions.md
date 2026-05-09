## atmanforge

> AtmanForge is a macOS/iOS image editor that integrates with AI models for image generation and manipulation. Users bring their own API keys (BYOK) — no backend or subscription required.

# AtmanForge

## Overview
AtmanForge is a macOS/iOS image editor that integrates with AI models for image generation and manipulation. Users bring their own API keys (BYOK) — no backend or subscription required.

## Tech Stack
- SwiftUI
- Xcode project (not SPM-based)
- Targets Apple platforms (macOS/iOS)

## Architecture
- `AtmanForge/` — main app source code
- `AtmanForge.xcodeproj` — Xcode project configuration

## Key Concepts
- **BYOK (Bring Your Own Keys):** Users configure their own API keys for AI providers. Keys are stored locally on-device.
- **AI-powered editing:** The app sends image editing requests to external AI model APIs and renders the results.

## Development
- Open `AtmanForge.xcodeproj` in Xcode to build and run.

---
> Source: [Nixarn/atmanforge](https://github.com/Nixarn/atmanforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
