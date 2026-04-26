## tasadezy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- **Dev server:** `npm run dev` (Vite, runs on localhost:5173)
- **Build:** `npm run build`
- **Lint:** `npm run lint`
- **Preview production build:** `npm run preview`
- **Install deps:** `npm install --legacy-peer-deps` (required due to eslint peer dep conflict)

## Architecture

Tasadezy Voices — a voice-over/audio samples showcase site built with React 19 + Vite 8, deployed on Vercel.

### Routing (React Router DOM v7)

- `/` → HomePage — hero, services (CardHoverEffect), workflow section with scroll-triggered animations, testimonials marquee
- `/demos` → DemosPage — audio sample gallery with language filters, waveform playback, manifest-driven (`/assets/new_voice_samples/audio-manifest.json`)
- `/contact` → ContactPage — form submitting to Google Forms via hidden iframe
- `*` → redirects to `/`

### Key Patterns

- **No TypeScript, no state management library** — all local state with hooks
- **Neomorphic design system** — soft shadows (`#d8d8d8`/`#ffffff`), rounded corners, inset pressed states. Background: `#f7f7f7`, accent pink `#c97b84`
- **Styling is mixed:** Tailwind CSS (loaded via CDN in index.html), large generated CSS file (`pages/global.css` ~260KB from Framer export), component-scoped CSS (`DemosPage.css`), and inline styles
- **Animations:** `motion` library (Framer Motion) for hover/scroll effects; CSS keyframes for waveform and marquee
- **`useLangColors` hook** (`lib/useLangColors.js`) — dynamically injects CSS for language-based color coding in DemosPage
- **`cn()` utility** (`lib/utils.js`) — clsx wrapper for conditional classnames

### Audio System

DemosPage fetches `audio-manifest.json` to populate samples. Each entry has: file path, title, language, voiceType, useCase, artist. Audio playback uses native `<audio>` with custom UI (play/pause, waveform visualization).

### Shared Layout

Header and Footer are imported directly by each page (not via a layout route). Header has both mobile and desktop nav variants with scroll-aware styling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YashSensei) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
