## ai-report-demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

This webapp serves as an AI Report for Matchlytics startup, displaying Padel game metrics and analytics. The application is designed to be served as a webview within a React application, providing comprehensive match analysis and visualization.

## Development Commands

- `yarn dev` - Start development server with Vite
- `yarn build` - TypeScript compile and production build (`tsc && vite build`)
- `yarn preview` - Preview production build locally
- `yarn format` - Format code with Prettier
- `yarn deploy` - Deploy to GitHub Pages

## Architecture

This is a React + TypeScript application built with Vite, focused on Padel match analytics visualization and reporting.

### Core Structure
- **Entry Point**: `src/main.tsx` renders the app with React 18's `createRoot`
- **Routing**: React Router with routes for entry page (`/`), game report (`/report/:playerId`), and 3D demo (`/3d`)
- **Main Components**:
  - `EntryPage` - Landing/selection page
  - `GameReport` - Main analytics dashboard
  - `Test` - 3D visualization demo
  - `BallMap/` - Complex tennis court visualization with heatmaps and ball tracking

### Key Features
- **BallMap Component**: Sophisticated Padel court visualization with multiple modes:
  - Ball hits tracking with shot type filtering
  - Player position heatmaps 
  - Animated transitions and fade effects
  - Uses heatmap.js for data visualization
- **3D Visualization**: Three.js integration via @react-three/fiber and @react-three/drei
- **Charts**: Multiple chart components (CircleChart, DualCircleChart, TripleCricleChart)
- **Padel Analytics**: Speed cards, serves analysis, player stats, match summaries

### Technical Configuration
- **Path Aliases**: `@/*` maps to `src/*`
- **Browser Support**: Legacy browser support with extensive polyfills for Chrome 61+, Firefox 60+, Safari 11+, Edge 16+
- **Styling**: Tailwind CSS with custom utilities
- **Linting**: ESLint with TypeScript, React, and Prettier integration
- **Build Target**: ES2015 with sourcemaps enabled

## Data Architecture

The application uses a robust, scalable data service architecture for handling backend integration:

### Data Service Layer
- **Main Service**: `src/services/dataService.ts` - Singleton service orchestrating all data operations
- **API Layer**: `src/services/api/statisticsApi.ts` - Handles loading statistics.json with error handling
- **Cache System**: `src/services/cache/cacheManager.ts` - Hybrid caching (memory + sessionStorage) with TTL support
- **Transformers**: `src/services/transformers/` - Convert raw backend data to component-ready format
- **Types**: `src/types/services.ts` - Service-specific TypeScript interfaces

### React Integration
- **Hooks**: `src/hooks/useMatchSummary.ts` - React hooks with loading/error states, retry logic, and caching
- **Components**: Modified to use data service instead of mock data
- **Error Handling**: Comprehensive error boundaries with retry capabilities

### Cache Strategy
- **Memory Cache**: 5-minute TTL for immediate access
- **Persistent Cache**: 30-minute TTL in sessionStorage (survives navigation)
- **LRU Eviction**: Prevents memory bloat with size limits
- **Automatic Invalidation**: Pattern-based cache clearing

### Backend Integration Status
- ✅ **Match Summary**: Migrated to real data (time in minutes, rally duration in seconds)
- 🔄 **Premium Features**: Shot Analysis, Serves Analysis, Points & Errors, Ball Hits (Pro overlay)
- 🔄 **Ready for Migration**: Speed Card, Distance Card (backend data available)

### Development Notes
- Uses custom polyfills for legacy browser compatibility
- Extensive TypeScript configuration with strict mode
- Components follow React functional patterns with hooks
- Global CSS utilities and Tailwind merge for class handling
- Data flows from statistics.json → API → Cache → Transformers → React Hooks → Components

---
> Source: [sdrrv/ai-report-demo](https://github.com/sdrrv/ai-report-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
