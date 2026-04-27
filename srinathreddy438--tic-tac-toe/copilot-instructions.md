## tic-tac-toe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-file, no-build web app. Everything lives in `index.html` — HTML structure, CSS styles, and JS game logic are all inline.

## Running the app

Open `index.html` directly in a browser:
```
open index.html
```

There is no build step, no package manager, and no dependencies.

## Git workflow

**After every change, commit and push to GitHub — no exceptions.** This ensures work is never lost and the remote always reflects the current state of the project.

- Remote: https://github.com/srinathreddy438/tic-tac-toe
- Branch: `main`
- Stage only the files that were actually changed
- Write a clean, descriptive commit message that explains *what* changed and *why*
- Run `git push` immediately after every commit — do not batch multiple commits before pushing

## Architecture

All game logic is contained within the `<script>` tag at the bottom of `index.html`:

- `board` — flat 9-element array representing the grid (`null` | `'X'` | `'O'`)
- `WINS` — hardcoded array of the 8 winning index triples
- `init()` — resets board state and re-renders; preserves `scores` across rounds
- `render()` — syncs DOM cells to `board` state and updates the scoreboard display
- `checkWinner()` — returns `{ winner, line }` on win, `{ winner: null, line: [] }` on draw, or `null` if game continues

Styles use a dark color scheme (`#1a1a2e` background, `#e94560` for X, `#a8dadc` for O).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/srinathreddy438) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
