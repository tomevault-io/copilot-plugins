## top-of-mind-extension

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Top of Mind is a Chrome Extension (Manifest V3) designed to help users remember things they find online. It uses React, Redux Toolkit, and TypeScript with a multi-context architecture.

## Key Commands

```bash
# Development
npm run watch    # Start development mode with file watching (no HMR)
npm run build    # Production build

# Code Quality
npm run typecheck  # Run TypeScript type checking
npm run lint       # Currently same as typecheck
```

## Architecture

### Multi-Context Extension Structure

The extension operates across multiple Chrome contexts with Redux state synchronization:

1. **Background Service Worker** (`src/apps/background/`): Hosts the main Redux store
2. **Popup** (`src/apps/popup/`): Browser action interface
3. **Side Panel** (`src/apps/sidepanel/`): Chrome's side panel interface  
4. **Quick Actions** (`src/apps/quickactions/`): Iframe-based quick actions

### State Management

- Redux Toolkit with webext-redux for cross-context synchronization
- Background script maintains the single source of truth
- UI contexts use proxy stores that auto-sync with the background store
- Store configuration: `src/store/store.ts`
- Current slices: `uiSlice` (theme, panels, compact mode)

### Component Organization

```
src/components/
├── atoms/      # Basic UI elements (Toggle)
├── containers/ # Smart components (BrowserContainer wraps all apps)
└── widgets/    # Composite UI components
```

### Important Patterns

1. **BrowserContainer**: All app entry points must be wrapped with `BrowserContainer` to connect to Redux
2. **Path Aliases**: Use `@/` for imports from `src/` directory
3. **TypeScript**: Strict mode enabled, all new code should be fully typed
4. **Routing**: Uses wouter with memory history in popup/sidepanel

## Working with Redux

When adding new features:
1. Create slices in `src/store/slices/`
2. Add to root reducer in `src/store/store.ts`
3. Actions automatically sync across all contexts via webext-redux
4. See `src/store/WEBEXT_REDUX_USAGE.md` for detailed patterns

## Chrome Extension APIs

- Manifest V3 service worker pattern
- Background script handles message passing and extension logic
- Content scripts not yet implemented but infrastructure exists
- Chrome types available via `@types/chrome`

## Development Notes

- No HMR support - requires extension reload after changes
- Redux DevTools enabled in development
- Tailwind CSS for styling
- Parcel bundler with webextension plugin
- No test infrastructure currently set up

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jhgaylor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jhgaylor)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
