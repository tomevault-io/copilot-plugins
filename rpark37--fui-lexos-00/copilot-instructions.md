## fui-lexos-00

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Angular 21 File Browser UI application using standalone components and signal-based reactive architecture. A modern file management interface with search, navigation, and hierarchical folder browsing.

## Commands

- `npm start` - Run development server (localhost:4200)
- `npm run build` - Production build
- `npm test` - Run Angular test suite
- `npm run watch` - Watch mode development build

## Architecture

### Signal-Based State Management

The app uses Angular 21's signal API for reactive state. Main state lives in `FileBrowserComponent` (`src/app/file-browser.component.ts`):

- `searchQuery`, `selectedFolder`, `activeRoute`, `viewMode`, `breadcrumbs`, `sidebarFolders`, `allFolders` - writeable signals
- `displayedFolders` - computed signal that auto-filters based on search query

### Component Communication

Components use signal-based inputs/outputs (`input()`, `output()`) instead of traditional decorators:

```typescript
// Input signal
folders = input.required<FolderItem[]>();
// Output signal
folderSelect = output<string>();
```

### Key Components

| Component | File | Purpose |
|-----------|------|---------|
| FileBrowserComponent | `file-browser.component.ts` | Main container, holds all state |
| TopBarComponent | `components/top-bar.component.ts` | Search bar + breadcrumb navigation |
| SidebarComponent | `components/sidebar.component.ts` | Collapsible folder tree |
| FolderGridComponent | `components/folder-grid.component.ts` | Card-based grid layout |
| BottomNavComponent | `components/bottom-nav.component.ts` | Bottom navigation bar |

### Design System

Design tokens in `src/app/design-tokens.ts`:
- Primary color: `#FF6B1A` (orange accent)
- All spacing, typography, and color constants defined here

### Styling Approach

- Component styles are inline CSS-in-JS (scoped)
- Global styles in `src/styles.css`
- Responsive breakpoints: mobile (<480px), tablet (481-768px), desktop (769-1024px), wide (>1024px)
- Uses Angular's new control flow syntax: `@if`, `@for`, `@switch`

### Data Interfaces

Core interfaces in `src/app/interfaces.ts`:
- `FolderItem` - folder/file data with optional children for tree structure
- `BreadcrumbItem` - navigation path items
- `NavItem` - bottom navigation items with badges

## Standalone Components Pattern

All components use `standalone: true` - no NgModule required:

```typescript
@Component({
  selector: 'app-name',
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `...`,
  styles: [`...`]
})
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rpark37)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rpark37)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
