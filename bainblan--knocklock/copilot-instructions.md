## knocklock

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Who's There is a smart door lock system that recognizes personalized knock patterns. It consists of two components:

1. **Next.js web app** (root `/app` directory) — the frontend/companion app
2. **SmartKnock** (`/SmartKnock` directory) — ESP32-C3 firmware for the physical knock sensor

Both components are early-stage (scaffolded but not yet built out).

## Commands

### Web App (Next.js)

```bash
npm run dev      # Start dev server (http://localhost:3000)
npm run build    # Production build
npm run start    # Start production server
npm run lint     # Run ESLint (eslint-config-next with core-web-vitals + typescript)
```

### ESP32 Firmware (SmartKnock)

Requires [PlatformIO](https://platformio.org/). From the `SmartKnock/` directory:

```bash
pio run                    # Build firmware
pio run --target upload    # Build and flash to ESP32-C3
pio device monitor         # Serial monitor
```

## Architecture

- **Web app**: Next.js 16 with App Router, React 19, TypeScript, Tailwind CSS v4 (via PostCSS plugin). Uses `@/*` path alias mapping to the project root. Fonts: Geist Sans and Geist Mono via `next/font/google`.
- **ESP32 firmware**: PlatformIO project targeting `esp32-c3-devkitm-1` with Arduino framework. Source lives in `SmartKnock/src/main.cpp`.
- The two components are independent — the web app uses npm, the firmware uses PlatformIO. They have separate `.gitignore` files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bainblan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
