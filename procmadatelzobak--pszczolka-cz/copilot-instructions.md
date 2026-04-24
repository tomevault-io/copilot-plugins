## pszczolka-cz

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal website for Vojtěch Pszczólka (pszczolka.cz) — an interactive diary-style SPA with a branching narrative. The visitor opens a "diary" and navigates through sections (WHO_AM_I, AI_TECH, TEAMBUILDING, WORLDS) by choosing paths. Visual style: dark cyberpunk × mystery, terminal fonts, gold accents, glitch effects.

The full concept document is in `pszczolka-concept-v2.md` (Czech language). Always consult it for content, visual specs, and branch structure.

## Architecture (from concept)

- **SPA** — single page, no router needed. Branches are JS/React state.
- **State model:** `{ node: 'INTRO', lang: 'cz', history: ['INTRO'] }` — history array enables back navigation.
- **i18n:** CZ (primary) and EN. Language stored in localStorage. Switching restarts current node.
- **Content structure:** `content.cz.INTRO.text`, `content.cz.INTRO.choices[]` etc.
- **Branch tree:** INTRO → KDO_JSEM, AI_TECH, TEAMBUILDING, SVĚTY (each with sub-branches).
- **Hosting target:** Static HTML/JS/CSS (GitHub Pages, Vercel, Netlify, or sinuhetcloud).

## Visual Specs

- Color palette defined as CSS custom properties (see concept §1): `--bg-primary: #0a0a0f`, `--text-primary: #c8b88a`, `--accent-gold: #d4a843`, `--accent-neon: #00ff9f`, `--accent-neon-alt: #ff2a6d`.
- Fonts: `IBM Plex Mono` / `JetBrains Mono` (body), `Cinzel` / `Cormorant Garamond` (headings).
- Typing effect: letter-by-letter at 40-60ms, pause on punctuation. Click anywhere to skip.
- Choices fade in after typing completes (600ms).
- Glitch: CSS RGB split, max 1-2× per page, subtle. Hover on choices triggers brief glitch.
- Noise overlay at 3% opacity, scanlines.
- Centered column layout, max-width ~650px, no navbar/footer.
- CZ|EN toggle in top-right corner.

## Key Constraints

- Respect `prefers-reduced-motion`: show all text immediately, no animations.
- Provide `<noscript>` fallback with full static content.
- Use `aria-live` for typed text (accessibility).
- Include OG meta tags and OG image.
- The name is always **Pszczólka** (with accent on ó).
- Tone: calm, quiet mystery. The diary offers paths — it doesn't scare or threaten.

## Priority Order

1. MVP: INTRO + 4 branch entry points (landing page)
2. V1: All CZ branches complete
3. V2: EN translation
4. V3: Glitch effects, sounds, OG image, SEO
5. V4: Easter eggs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/procmadatelzobak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
