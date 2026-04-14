## equity-calc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Texas Hold'em poker equity calculator with visual card selection and poker table interface. Client-side only implementation for 2-6 players.

## Technology Stack

- **HTML/CSS/JavaScript**: Pure vanilla implementation, no frameworks
- **Mobile-first**: Responsive design for touch interfaces
- **Client-side only**: No server dependencies

## Development Commands

```bash
# Serve locally (any simple HTTP server)
python -m http.server 8000
# or
npx serve .
```

## Architecture

### Core Components

1. **Card System** (`cards.js`):
   - 52-card deck representation
   - Card selection and validation logic
   - Visual card components

2. **Poker Table UI** (`table.js`):
   - Visual poker table layout
   - Player position management (2-6 players)
   - Community cards area

3. **Equity Calculator** (`equity.js`):
   - Monte Carlo simulation engine
   - Hand evaluation algorithm
   - Win probability calculations

4. **UI Controllers** (`app.js`):
   - Card selection interactions
   - Player management
   - Results display

### File Structure
```
index.html          # Main application
styles.css          # Poker table styling, mobile-responsive
js/
  ├── app.js         # Main application controller
  ├── cards.js       # Card deck and selection logic
  ├── table.js       # Poker table UI components
  ├── equity.js      # Equity calculation engine
  └── utils.js       # Helper functions
```

## Key Features

- **Visual Card Selection**: Touch-friendly card picker
- **Poker Table Layout**: Circular table with player positions
- **Real-time Equity**: Updates as cards are selected
- **Mobile Responsive**: Works on phones and tablets
- **2-6 Players**: Dynamic player count adjustment

## Development Guidelines

- Keep JavaScript vanilla (no jQuery, React, etc.)
- Use CSS Grid/Flexbox for layouts
- Touch-first mobile interactions
- Fast equity calculations (< 1 second)
- Clear visual feedback for card selections

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dlqs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dlqs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
