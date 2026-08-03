## wake-up-clock

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a wake-up clock web application designed for a 5-year-old child. It's a single-page application (index.html) that shows when different activities become available throughout the morning.

## Commands

- `npm start` - Starts a local Python HTTP server on port 8000
- `npm run build` - Copies index.html to dist/ folder for GitHub Pages deployment
- `npm run deploy` - Runs build and displays deployment ready message

## Architecture

The entire application is contained in a single `index.html` file with:
- Inline CSS for styling
- Inline JavaScript for functionality
- No external dependencies or build process
- SVG icons embedded directly in the HTML

## Key Features and Schedule

Activities unlock at specific times:
- YouTube: 6:00 AM
- Drawing: 6:00 AM  
- Minecraft: 6:30 AM
- Get up: 7:00 AM

Before unlock times, activities show countdown in minutes. After unlock, they show a green checkmark (✓).

## Testing Time Simulation

Use the `?time=HH:MM` query parameter to simulate different times:
- Example: `http://localhost:8000?time=05:45` simulates 5:45 AM

## Voice Features

- Uses Web Speech API with configurable voice selection
- Prefers male English voices (Alex, Daniel, Oliver, etc.)
- Speech patterns: 
  - Time: "The time is... X oh Y AM/PM"
  - Countdown: "X minutes until you can [activity]"
  - Available: "You can [activity]!"

## Animations

Click-triggered animations (0.5s duration):
- YouTube: rotate
- Drawing: wiggle
- Minecraft: mining (custom pickaxe bashing motion)
- Get up: bounce

---
> Source: [Thomas101/wake-up-clock](https://github.com/Thomas101/wake-up-clock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
