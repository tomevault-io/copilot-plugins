## cirklon

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Cirklon emulator built with Astro and TypeScript, recreating the functionality of the Sequentix Cirklon hardware sequencer/synthesizer in the browser using Web Audio API, Tone.js, and Pleco-xa.

## Commands

- **Development**: `npm run dev` (start dev server - avoid running this as it causes crashes, let user start in another terminal)
- **Build**: `npm run build`
- **Preview**: `npm run preview`

## Architecture

### Audio Stack
- **Web Audio API**: Core browser audio processing
- **Tone.js**: High-level audio synthesis and sequencing framework
- **Pleco-xa**: Additional audio processing utilities

### Frontend Stack
- **Astro**: Static site generator with component islands
- **TypeScript**: Type-safe development with strict mode
- **WebAudio lib**: TypeScript definitions for Web Audio API

## Development Environment

- TypeScript configured with ES2022 target and WebAudio library support
- ESModule resolution for modern JavaScript features
- Audio-specific TypeScript configuration for Web Audio API types

## Important Notes

- Never start dev servers directly (npm run dev, astro dev) - they cause crashes
- Always inform user when server needs to be started so they can do it in another terminal
- Audio context requires user interaction to start (Web Audio API security requirement)

## Asset Organization

- **IMAGES/**: Contains Cirklon hardware reference images and design assets
  - Product photography for UI reference
  - Vector graphics and logos
  - Design source files (PSD, AI)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brookcs3) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
