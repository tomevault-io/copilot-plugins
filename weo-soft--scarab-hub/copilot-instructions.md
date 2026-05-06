## specify-rules

> ﻿# scarab-hub Development Guidelines

﻿# scarab-hub Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-12-27

## Active Technologies
- JavaScript (ES6 modules) + Vite 5.0.0, Vitest 1.0.0 (001-additional-item-prices)
- LocalStorage (browser), JSON files in `/public/data` (001-additional-item-prices)
- JavaScript (ES6+), HTML5, CSS3 + Vite (build tool and dev server), existing codebase infrastructure (001-essence-reroll)
- LocalStorage (browser) for user preferences and cached market data, same as Scarab feature (001-essence-reroll)
- LocalStorage (browser) for user preferences and cached market data, same as Essence feature (001-fossil-reroll)
- JavaScript (ES6+), HTML5, CSS3 + Vanilla JavaScript (no external frameworks), Canvas API for grid views (if needed) (003-temple-upgrade-display)
- Static JSON files in `/public/data/items/`, images in `/public/assets/images/` (003-temple-upgrade-display)
- JavaScript (ES6+), HTML5, CSS3 + Vite (build and dev server), existing codebase (generic grid, list view, adapters) (001-regex-item-search)
- In-memory selection state per category; optional LocalStorage for persistence (out of scope per spec unless added later) (001-regex-item-search)
- JavaScript (ES6+), HTML5, CSS3 + Vite 5.0.0 (build tool), Vitest 1.0.0 (testing), Canvas API (for grid views), existing codebase infrastructure (005-mobile-responsive)
- N/A (responsive design is presentation-layer only, no data storage changes) (005-mobile-responsive)

- JavaScript (ES6+), HTML5, CSS3 + Vite (build tool and dev server), minimal external libraries (001-flipping-scarabs)

## Project Structure

```text
backend/
frontend/
tests/
```

## Commands

npm test; npm run lint

## Code Style

JavaScript (ES6+), HTML5, CSS3: Follow standard conventions

## Recent Changes
- 005-mobile-responsive: Added JavaScript (ES6+), HTML5, CSS3 + Vite 5.0.0 (build tool), Vitest 1.0.0 (testing), Canvas API (for grid views), existing codebase infrastructure
- 001-regex-item-search: Added JavaScript (ES6+), HTML5, CSS3 + Vite (build and dev server), existing codebase (generic grid, list view, adapters)
- 003-temple-upgrade-display: Added JavaScript (ES6+), HTML5, CSS3 + Vanilla JavaScript (no external frameworks), Canvas API for grid views (if needed)


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Source: [weo-soft/scarab-hub](https://github.com/weo-soft/scarab-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
