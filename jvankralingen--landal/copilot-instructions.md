## landal

> This is a React 18 application for **Landal Werkenbij** - a localized content generator for job profiles across multiple Landal holiday parks. The app generates park-specific job descriptions based on each park's unique "vibe" (atmosphere/theme).

# CLAUDE.md

## Project Overview

This is a React 18 application for **Landal Werkenbij** - a localized content generator for job profiles across multiple Landal holiday parks. The app generates park-specific job descriptions based on each park's unique "vibe" (atmosphere/theme).

## Tech Stack

- **React 18** with functional components and hooks
- **Create React App** (react-scripts) for tooling
- No additional UI libraries - uses inline styles throughout

## Commands

```bash
npm start    # Start development server
npm run build # Production build
npm test     # Run tests (Jest with jsdom)
```

## Project Structure

```
src/
├── App.js                    # Main application component
├── index.js                  # Entry point
├── components/
│   ├── PreviewModal.js       # Modal for previewing generated content
│   ├── ParkSelector.js       # Park selection accordion/list component
│   └── BrandAssets.js        # Brand visual elements (icons, shapes)
└── data/
    ├── constants.js          # Brand colors, fonts, vibe configurations
    ├── parkData.js           # Park database organized by country/region
    └── jobProfiles.js        # Job profile definitions (functiefamilies)
```

## Key Concepts

### Vibes (Park Atmospheres)
Parks are categorized by vibe/atmosphere type defined in `constants.js`:
- `luxe` - Premium
- `kinderen` - Kindvriendelijk (Kid-friendly)
- `natuur` - Nature
- `strand` - Coastal
- `actief` - Active/Adventure
- `wellness` - Wellness
- `familie` - Family
- `watersport` - Water sports

### Localization
Content is generated per park based on:
1. Selected job profile (functieprofiel)
2. Park's vibe configuration
3. Park-specific tags/highlights

### State Management
All state is managed via React useState hooks in App.js:
- `selectedParks` - Array of selected park IDs
- `generatedContent` - Object mapping parkId to generated content
- `selectedFunctie` - Currently selected job profile
- Theme state (dark/light mode)

## Coding Patterns

- **Inline styles**: All styling uses inline style objects with theme-aware colors
- **Dutch naming**: Code comments and some variables are in Dutch (e.g., `functiefamilies`, `beschrijving`)
- **Simulated AI**: Content generation simulates an API delay with setTimeout (1.5s)

## Brand Constants

Primary brand colors are defined in `BRAND.colors`:
- Teal: `#0097a2` (primary accent)
- Sand: `#fffae9` (background)
- Amber: `#e78402`
- Forest: `#30ae2b`

---
> Source: [jvankralingen/landal](https://github.com/jvankralingen/landal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
