## github-pr-widget

> A desktop Electron application that displays GitHub pull requests in a minimalist widget interface. Features include minimize/expand functionality, theme customization, status indicators, and auto-resizing.

# GitHub PR Widget - Claude Context

## Project Overview
A desktop Electron application that displays GitHub pull requests in a minimalist widget interface. Features include minimize/expand functionality, theme customization, status indicators, and auto-resizing.

## Architecture

### Core Components
- **PRRenderer.ts**: Handles rendering individual PR items with click event management
- **renderer.ts**: Main widget logic, state management, and UI coordination
- **GithubProvider.ts**: GitHub API integration (referenced but not examined)
- **styles.css**: CSS styling and animations

### State Management
The application persists state across sessions using multiple storage mechanisms:

1. **Minimize State** (`MinimizeStateManager` - lines 91-137 in renderer.ts)
   - Uses browser `localStorage` with key `'minimized-prs'`
   - Stores array of PR IDs that are minimized
   - Loaded on startup, saved on every toggle

2. **Theme Settings** (`ThemeManager` - lines 139-207 in renderer.ts)
   - Uses Electron IPC calls to main process
   - `get-theme` and `set-theme` IPC channels
   - Supports customizable colors, gradients, and design tokens

3. **GitHub Token** (lines 308+ in renderer.ts)
   - Stored via Electron main process (secure storage)
   - Loaded via `get-github-token` IPC call

4. **Window Dimensions**
   - Managed by Electron main process
   - Auto-resize functionality calculates optimal height based on content

## Recent Bug Fixes

### PR Title Click Issue (Fixed)
**Problem**: When a PR row started minimized but was then expanded, clicking the PR title would minimize the row instead of opening the PR URL.

**Root Cause**: The click handler in `PRRenderer.ts` was using a stale `isMinimized` parameter from initial render instead of checking the current visual state.

**Solution**: Modified click handler (lines 131-155 in PRRenderer.ts) to check current state:
```typescript
const currentlyMinimized = prItem.classList.contains('minimized');
```

**Key Learning**: When elements can change state after initial render, always check current DOM state rather than relying on render-time parameters.

## Design Decisions

### Animation & UX
- Smooth minimize/expand transitions using CSS grid and transforms
- Staggered animation delays for PR list items
- Auto-resize window based on content height
- Preserve user manual window sizing while ensuring content visibility

### State Persistence Strategy
- **localStorage**: UI state (minimize/expand) - fast, client-side
- **Electron Store**: Settings and credentials - secure, persistent across updates
- **No caching**: PR data fetched fresh from GitHub API on each startup

### Click Event Handling
- Event delegation for dynamic content (tooltips, status indicators)
- Careful event propagation management for nested clickable elements
- Real-time state checking for elements that change after render

## Technical Notes

### Auto-Resize Logic
- `calculateOptimalHeight()`: Accounts for minimized vs expanded items
- `autoFitHeight()`: Conservative resize (only if content needs more space)
- `manualAutoFitHeight()`: Always resize to optimal (triggered by double-click)

### Theme System
- CSS custom properties for dynamic theming
- HSL color manipulation for programmatic color variations
- Design tokens for consistent spacing and dimensions

## Development Guidelines

### State Management
- Always persist UI state that affects user workflow
- Use appropriate storage mechanism (localStorage vs Electron store)
- Check current DOM state for elements that change after render

### Event Handling
- Use event delegation for dynamic content
- Be explicit about event propagation (stopPropagation when needed)
- Test click behavior thoroughly for nested interactive elements

### Performance
- Silent refresh for background updates (no loading animation)
- Only re-render when data actually changes
- Use requestAnimationFrame for DOM measurements and updates

---
> Source: [blakehaddad/github-pr-widget](https://github.com/blakehaddad/github-pr-widget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
