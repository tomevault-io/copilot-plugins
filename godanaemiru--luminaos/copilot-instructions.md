## luminaos

> LuminaOS is a React-based desktop OS simulator featuring draggable windows, a virtual file system, and multiple built-in apps like Terminal, File Explorer, and Paint.

# LuminaOS Agent Instructions

## Project Overview
LuminaOS is a React-based desktop OS simulator featuring draggable windows, a virtual file system, and multiple built-in apps like Terminal, File Explorer, and Paint.

## Build and Run Commands
- Development: `npm run dev` (starts Vite dev server with hot reload)
- Build: `npm run build`
- Lint: `npm run lint`

## Architecture Decisions
- **State Management**: Local useState hooks in components; no global state library
- **Component Structure**: App.jsx manages window lifecycle; Window.jsx wraps each app with dragging/maximize/close
- **Virtual File System**: Simple array of objects in localStorage for custom files/folders
- **Persistence**: localStorage for custom apps and shortcuts

## Project-Specific Conventions
- Component exports: Named exports with PascalCase
- App IDs: kebab-case strings
- Theme access: Via `osProps.theme` prop
- VFS navigation: Filter by `parentId`
- Persistence keys: Prefixed with `lumina_`

## Potential Pitfalls
- Window positions reset on close (by design)
- Ensure all VFS items have `parentId` for navigation — `createFolder`/`createFile` take `parentId` as their first argument, so never pass them directly as an `onClick` handler
- A crashing app is contained by `ErrorBoundary` in Window.jsx rather than taking down the desktop
- localStorage writes are wrapped in try/catch; a full quota logs a warning and drops the write

## Key Files and Directories
- [src/App.jsx](src/App.jsx): Core OS logic and app registry
- [src/components/Window.jsx](src/components/Window.jsx): Reusable window component (drag, maximize, viewport clamping)
- [src/components/ErrorBoundary.jsx](src/components/ErrorBoundary.jsx): Per-app crash containment
- [src/apps/](src/apps/): Individual app components
- [src/config/constants.js](src/config/constants.js): Theme and hardcoded data
- [README.md](README.md): Basic setup instructions

## Development Tips
- Use React DevTools to inspect component tree
- Check localStorage in DevTools for persisted data
- Apps receive `osProps` with theme, vfs, and callbacks</content>
<parameter name="filePath">c:\dev\luminaos\AGENTS.md

---
> Source: [godanaemiru/luminaos](https://github.com/godanaemiru/luminaos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
