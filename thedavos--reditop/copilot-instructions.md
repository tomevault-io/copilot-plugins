## reditop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Reditop is a Vue.js 2 single-page application that visualizes popular Reddit posts from r/all. It's built with TypeScript, Vite, Pinia state management, and Tailwind CSS.

## Common Development Commands

```bash
# Development
npm run dev              # Start development server with hot reload
npm run build           # Production build with type checking
npm run preview         # Preview production build

# Testing
npm run test:unit       # Run Vitest unit tests
npm run test:e2e        # Run Cypress E2E tests

# Code Quality  
npm run lint            # ESLint code quality checks
```

## Architecture Overview

### Component Hierarchy
- **Base UI Components**: Custom design system (`UButton`, `UCard*`, `UDialog*` families) built due to Vue 2 incompatibility with modern UI libraries
- **Domain Components**: Reddit-specific components (`UPostItem`, `UPostDetail`, `UPostError`)
- **Single-page Architecture**: All functionality in `src/App.vue` with component composition

### State Management (Pinia)
- **Store Location**: `src/stores/reddit.store.ts`
- **Persistent State**: Uses `@vueuse/core`'s `useLocalStorage` for persistence between sessions
- **Key State**: `posts[]`, `postStates{}` (user interactions), `selectedPost`, `currentPage`, pagination tokens
- **Important Actions**: `loadPosts()`, `selectPost()`, `dismissPost()`, `resetAndReload()`

### API Integration
- **Reddit API**: Fetches from `r/all/top.json` via CORS proxy (`cors-anywhere.com`)
- **Service Layer**: `src/services/reddit.api.ts` handles all external requests
- **Type Safety**: Comprehensive interfaces in `src/types/reddit.ts`

### Directory Structure
```
src/
├── components/         # UI components (card/, dialog/, posts/, base components)
├── composables/        # Reusable composition functions  
├── services/           # API services and external integrations
├── stores/             # Pinia state management
├── types/              # TypeScript type definitions
├── utils/              # Utility functions (className, date, image processing)
└── router/             # Vue Router configuration
```

### Build Configuration
- **Vite**: Uses `@vitejs/plugin-vue2` for Vue 2 support
- **TypeScript**: Project references pattern with separate configs for app, config, and testing
- **Target**: ES2022 with legacy browser support (Chrome 91+, Firefox 88+, Safari 15+)
- **Path Alias**: `@/` points to `/src`

### Styling System
- **Tailwind CSS 3**: Custom design system with Reddit-specific colors (`orange: #ff4500`, `blue: #0079d3`)
- **Custom Animations**: Post dismissal and UI transitions defined in `tailwind.config.ts`
- **Responsive**: Mobile-first approach with sidebar navigation

### Key Patterns
- **Client-side Pagination**: 10 posts per page over server-fetched data
- **Optimistic UI**: Immediate feedback for user interactions (mark as read, dismiss)
- **State-based Rendering**: Components react to Pinia store state changes
- **Utility Functions**: Centralized in `src/utils/` for image processing, formatting, CSS class merging

### Testing Setup
- **Unit Tests**: Vitest with jsdom environment
- **E2E Tests**: Cypress for critical user flows
- **Test Files**: Co-located with components or in dedicated test directories

## Important Notes
- Uses Vue 2.7 with Composition API support (not Vue 3)
- Custom UI components required due to library compatibility constraints
- State persists between sessions via localStorage
- CORS proxy required for Reddit API access in browser environment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thedavos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thedavos)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
