## mengmeng-pangpang

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a romantic commemorative website called "蒙蒙和胖胖的小窝" (Mengmeng and Pangpang's Home) - a single-page HTML application that celebrates a couple's relationship milestone. The site displays relationship information and calculates the number of days since they first met.

## Architecture

This is a simple static website consisting of a single HTML file with embedded CSS and JavaScript:

- **index.html**: Complete standalone web page with inline styles and scripts
- No build system, package.json, or external dependencies
- Pure HTML/CSS/JavaScript implementation

## Key Features

The website includes:
- Animated gradient background
- Floating heart animations and decorative elements
- Relationship milestone information display
- Automatic day counter calculation (calculates days since 2025-05-18)
- Interactive click effects that generate floating hearts
- Heart-shaped particle effects and meteor animations
- Responsive design for mobile devices
- Accessibility support with reduced motion preferences

## JavaScript Functionality

Core functions include:
- `calculateDays()`: Calculates and animates the day counter from the start date (2025-05-18)
- `animateNumber()`: Provides smooth number transition animations
- `createFloatingHeart()`: Generates random floating heart elements
- `createHeartParticles()`: Creates heart-shaped particle path animations
- `createHeartMeteor()`: Implements meteor trail effects with hearts
- `addClickEffects()`: Handles user interaction effects

## Development Notes

- All content is in Chinese (Simplified)
- Uses modern CSS features including CSS Grid, Flexbox, and CSS animations
- Implements accessibility features with `prefers-reduced-motion` media query
- Mobile-responsive with breakpoint at 768px
- No external dependencies or build process required
- Start date is hardcoded as May 18, 2025

## Running the Project

Simply open `index.html` in any modern web browser. No build process or server required.

---
> Source: [snowden-fu/mengmeng-pangpang](https://github.com/snowden-fu/mengmeng-pangpang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
