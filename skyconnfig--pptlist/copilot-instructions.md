## pptlist

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development Commands
```bash
npm install          # Install dependencies
npm run dev         # Start development server (http://127.0.0.1:5173/)
npm run build       # Build for production (runs type-check and build-only)
npm run build-only  # Build without type checking
npm run type-check  # Run TypeScript type checking
npm run lint        # Run ESLint with auto-fix
npm run preview     # Preview built application
```

### Code Quality
- **Linting**: `npm run lint` - Uses ESLint with Vue 3 + TypeScript configuration
- **Type Checking**: `npm run type-check` - Uses vue-tsc for TypeScript validation
- The project uses strict ESLint rules including single quotes, no semicolons, 2-space indentation

## Architecture Overview

### Technology Stack
- **Framework**: Vue 3 with Composition API and TypeScript
- **State Management**: Pinia stores
- **Build Tool**: Vite with Vue plugin
- **Styling**: SCSS with global variables and mixins
- **Rich Text**: ProseMirror for text editing
- **Icons**: @icon-park/vue-next
- **Database**: IndexedDB (via Dexie) for local data persistence

### Application Structure
PPTist is a web-based PowerPoint-like presentation editor with three main views:

1. **Editor View** (`src/views/Editor/`): Main editing interface for desktop
2. **Screen View** (`src/views/Screen/`): Presentation/slideshow mode
3. **Mobile View** (`src/views/Mobile/`): Mobile-optimized editor and player

### Core Stores (Pinia)
- `useMainStore`: Application state, database management
- `useSlidesStore`: Slide data and operations
- `useScreenStore`: Presentation mode state
- `useSnapshotStore`: Undo/redo functionality via IndexedDB
- `useKeyboardStore`: Keyboard shortcut state

### Key Directories
- `src/components/`: Reusable UI components (Button, ColorPicker, Modal, etc.)
- `src/views/Editor/Canvas/`: Canvas rendering and element manipulation
- `src/views/Editor/Toolbar/`: Element styling panels and tools
- `src/hooks/`: Composable functions for business logic
- `src/configs/`: Configuration files (fonts, themes, hotkeys, etc.)
- `src/utils/`: Utility functions
- `src/types/`: TypeScript type definitions

### Element System
The application supports various slide elements:
- **Text Elements**: Rich text with ProseMirror
- **Image Elements**: With cropping, filters, and effects
- **Shape Elements**: SVG-based shapes with customizable properties
- **Chart Elements**: Integration with ECharts
- **Table Elements**: Spreadsheet-like functionality
- **Line Elements**: Various line types and styles
- **Video/Audio Elements**: Media playbook
- **LaTeX Elements**: Mathematical formulas

### Canvas Architecture
- **Canvas Component**: Main editing surface with viewport management
- **Element Operations**: Drag, resize, rotate, align operations
- **Selection System**: Single and multi-element selection
- **Alignment System**: Magnetic alignment and distribution tools

### Data Flow
1. Mock data loaded from `src/services/` on app initialization
2. Slides stored in Pinia store and persisted to IndexedDB
3. Snapshot system tracks changes for undo/redo
4. Export functionality supports PPTX, PDF, images, and JSON formats

### Styling Conventions
- Uses SCSS with global variables in `src/assets/styles/variable.scss`
- Mixins available in `src/assets/styles/mixin.scss`
- Component-scoped styles with Vue's `<style lang="scss" scoped>`
- CSS naming follows BEM-like conventions

### API Integration
- Development proxy: `/api` routes proxy to `https://server.pptist.cn`
- Mock data system for offline development
- AI PPT generation features (requires backend integration)

### Mobile Considerations
- Responsive design with separate mobile views
- Touch-optimized interactions for mobile devices
- Simplified toolbar and editing experience on mobile

## Development Guidelines

### Component Development
- Follow Vue 3 Composition API patterns
- Use TypeScript with strict type checking
- Implement proper prop validation and emission typing
- Follow existing component structure and naming conventions

### State Management
- Use Pinia stores for global state
- Keep component-local state minimal
- Use `storeToRefs()` for reactive store properties
- Implement proper store actions for state mutations

### Styling
- Use SCSS variables for consistent theming
- Follow existing component styling patterns
- Ensure responsive design for mobile compatibility
- Use CSS Grid and Flexbox for layouts

### Performance Considerations
- Implement proper element virtualization for large presentations
- Use computed properties for expensive calculations
- Optimize canvas rendering for smooth interactions
- Implement proper cleanup in component unmount hooks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skyconnfig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
