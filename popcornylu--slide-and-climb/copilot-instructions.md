## slide-and-climb

> A web-based Snakes and Ladders board game, deployed on GitHub Pages.

# 滑梯棋 (Slide and Climb)

## Project Overview
A web-based Snakes and Ladders board game, deployed on GitHub Pages.

## Tech Stack
- Vanilla HTML/CSS/JavaScript (no frameworks)
- Web Audio API for sound effects
- Canvas for spinner rendering

## Project Structure
- `index.html` - Main game page, version displayed in `#version` div
- `css/style.css` - Styles
- `js/main.js` - Game logic, player setup, turn management, release notes UI
- `js/board.js` - Board rendering, cell colors, movement animation
- `js/spinner.js` - Spinner canvas with segment highlighting/dimming
- `js/sound.js` - Audio system (file-based with Web Audio API fallback)
- `js/release-notes.js` - Release notes data (viewable in-game by clicking version)
- `sounds/` - Audio files (tick.mp3, result.mp3, step.mp3, ladder.mp3)

## Key Rules
- P1 is always a human player (not changeable)
- Turn order is randomly shuffled (not spinner-based)
- Ladder start cells have green number text, end cells have green background
- Snake start cells have red number text, end cells have red background

## Version & Release Notes
- Current: v1.9.0
- Format: semver, displayed in index.html `#version` div
- Release notes viewable in-game by clicking the version number (bottom-right)
- Release notes data stored in `js/release-notes.js`

## Deploy Workflow (MUST follow on every deploy)
1. Bump version in `index.html` (`#version` div)
2. Update all `?v=` cache bust query strings in `index.html` (CSS and JS references) to match new version
3. Add new entry at the TOP of `ReleaseNotes` array in `js/release-notes.js` with:
   - `version`: new version string (e.g. 'v1.8.0')
   - `date`: today's date (YYYY-MM-DD)
   - `notes`: array of change descriptions in Chinese
4. Update `CLAUDE.md` version number (this file, "Current:" line above)
5. `git add` changed files, commit, and `git push origin main`

## Deployment
- `git push origin main` triggers GitHub Pages deployment
- URL: https://popcornylu.github.io/slide-and-climb/

## Sound Effects
- Audio files in `sounds/` directory (mp3 preferred for browser compatibility)
- Fallback chain: .mp3 -> .wav -> .aiff
- AIFF files may not work in Chrome; convert to MP3 with ffmpeg

## Development
- Use `python3 -m http.server 8000` for local testing
- Hard refresh (Cmd+Shift+R) to bypass cache

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/popcornylu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
