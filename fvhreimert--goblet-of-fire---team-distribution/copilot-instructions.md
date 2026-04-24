## goblet-of-fire-team-distribution

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Harry Potter "Goblet of Fire" themed team generator - a single-page web application that distributes names into teams with theatrical visual effects. Users enter a list of names and specify the number of teams, then names are animated as "notes" flying from a central goblet to team containers.

## Development

No build tools or package manager - this is a vanilla HTML/CSS/JS project. To run locally, serve the files with any static file server (e.g., `npx serve`, `python -m http.server`, or open `index.html` directly in a browser).

## Architecture

**index.html** - Entry point. Loads GSAP (with Physics2D and MotionPath plugins) from CDN, FontAwesome for icons, and links to local CSS/JS. Contains the video background element, goblet, controls form, teams container, and restart button.

**app.js** - All application logic:
- `startDistribution()` - Main function triggered by start button. Validates input, creates team containers, shuffles names, and schedules timed animations for each name.
- `distributeName(name, teamDiv, isLast)` - Animates a single note from center screen to target team using GSAP timeline. Handles cleanup and final sound on last note.
- `flashNoteEffectEnhanced(centerX, centerY, container)` - Creates the visual effect burst (flash, shockwave, sparks, gold particles) when a note appears. Uses GSAP Physics2D plugin for spark physics.

**styles.css** - All styling including:
- Custom `HarryPotter` font face (requires `harry_potter_font.ttf`)
- Video background positioning and z-index layering
- Effect elements (`.big-flash`, `.shockwave`, `.spark`, `.gold-spark`, `.magic-beam`)
- Note styling with `note.png` background image
- Responsive breakpoints at 768px and 500px

## Key Technical Details

- GSAP handles all animations; Physics2DPlugin provides gravity-based spark physics
- Z-index layering: video (-2) < goblet (1) < effects (5) < teams (15) < controls (20) < restart button (25) < notes during animation (30)
- Cache-busting via `?v=1.2` query params on CSS/JS includes
- Audio: `soundeffect.mp3` plays before each note, `final_sound_effect.mp3` on completion
- Background video auto-plays muted, unmutes on first user click

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fvhreimert) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
