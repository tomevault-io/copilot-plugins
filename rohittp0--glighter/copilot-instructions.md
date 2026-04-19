## glighter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Glighter** is a Progressive Web App (PWA) for creating map animations and exporting them as videos. Users select locations on a map, choose an animation template, preview the animation, and export it as a downloadable video, the export should happen on the client itself.

### Core Features
- Location selection via interactive map with numbered, draggable markers
- Template selection with split-screen preview (template on right, live map animation on left)
- Video export of map animations
- Mobile-first, native-like UI/UX
- Offline-capable PWA (installable with maximum offline functionality)
- MapLibre GL JS for map rendering
- MapTiler for tiles and styles

### User Flow
1. **Location Selection Screen**: Click map to add numbered markers, drag to reposition, delete unwanted markers
2. **Template Selection Screen**: Split view showing templates (right) and live animation preview (left)
3. **Export**: Generate and download animation as video file, show a progress screen while video is exporting

## Development Commands

```bash
# Start development server with HMR
npm run dev

# Type-check and build for production
npm run build

# Preview production build locally
npm run preview

# Run ESLint for code quality
npm run lint
```

## Tech Stack

- **Framework**: React 19 + TypeScript
- **Build Tool**: Vite 7
- **Maps**: MapLibre GL JS (not yet installed)
- **Tile Provider**: MapTiler (API key required)
- **Styling**: CSS (currently vanilla, consider Tailwind CSS or styled-components for production)

## Architecture Notes

### Map Implementation
- Use MapLibre GL JS for all map interactions
- MapTiler will provide both base map tiles and map styles
- Store MapTiler API key in environment variables (`.env.local`), never commit it
- Marker management should support:
  - Click-to-add with sequential numbering
  - Drag-and-drop repositioning
  - Delete functionality
  - State persistence during navigation

### PWA Requirements
- Add PWA manifest and service worker configuration to Vite
- Configure service worker to cache:
  - App shell (HTML, CSS, JS)
  - Map tiles for offline viewing (cache strategy)
  - Animation templates
- Handle offline/online state transitions gracefully
- Show appropriate error states when network is required but unavailable

### Video Export
- Research and integrate a library for recording canvas/map animations (e.g., MediaRecorder API, canvas-to-video libraries)
- Consider performance implications of video encoding on mobile devices
- Provide progress indicator during export
- Handle export failures with clear error messaging

### State Management
- Location data (marker positions, order)
- Selected template
- Export settings
- Offline/online status
- Consider using React Context or a lightweight state library (Zustand, Jotai) as complexity grows

### Mobile-First Design
- Responsive layouts with mobile as primary target
- Touch-friendly UI elements (minimum 44px touch targets)
- Smooth transitions and animations
- Consider gesture controls for map interactions
- Test on actual mobile devices, not just browser dev tools

## Important Considerations

- **Error Handling**: All async operations (map loading, tile fetching, video export) need comprehensive error handling
- **Performance**: Map rendering and video encoding are resource-intensive; optimize for mobile
- **API Keys**: Always use environment variables for MapTiler API key
- **Browser Compatibility**: Test PWA features across browsers (Safari, Chrome, Firefox mobile)
- **Accessibility**: Ensure map controls and UI are keyboard navigable where possible

## File Structure (Expected)

```
src/
├── components/        # Reusable UI components
├── screens/          # Main app screens (LocationSelection, TemplateSelection, Export)
├── hooks/            # Custom React hooks (useMap, useMarkers, useVideoExport)
├── utils/            # Helper functions
├── types/            # TypeScript type definitions
├── styles/           # Global styles and theme
└── services/         # External service integrations (MapTiler, video export)
```

## Configuration Files

- **vite.config.ts**: Vite configuration (add PWA plugin here)
- **tsconfig.json**: TypeScript configuration with project references
- **eslint.config.js**: Flat ESLint config with React hooks and refresh plugins
- **public/**: Static assets, PWA manifest, icons

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rohittp0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
