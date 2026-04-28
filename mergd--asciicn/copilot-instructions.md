## asciicn

> This is a Next.js 15 project for a UI library that is shadcn CLI compatible. The aesthetic is **TUI-inspired** (Terminal User Interface) - bare-bones, minimal, and functional with ASCII/Unicode character components.


## Project Overview

This is a Next.js 15 project for a UI library that is shadcn CLI compatible. The aesthetic is **TUI-inspired** (Terminal User Interface) - bare-bones, minimal, and functional with ASCII/Unicode character components.

We want to create authentic terminal-style UI components that work on modern websites.

## Design Philosophy

### TUI Aesthetic First

- **Look and feel like a terminal application** - not a modern web app with ASCII decorations
- Use bracket notation for interactive elements: `[x]`, `[ok]`, `[cancel]`, `[^k]`
- **Lowercase text everywhere** - feels more unix/terminal-like
- No fancy styling - let ASCII characters do the work
- Components should feel like they belong in a TUI, not a GUI

### ASCII/Unicode Components

- Build UI with **ASCII and Unicode characters** for borders, icons, backgrounds
- Use Unicode block characters (░▒▓█) for textures and backgrounds, not just loading states
- Store all components in `registry/ascii/` (single source of truth)
- Import components via `@/registry/ascii/[component]`
- Components should support **variants** (e.g., single/double borders) - NO rounded borders
- Keep it minimal and authentic - this is a TUI, not decoration

### Typography & Text

- **Monospace font** for the entire site (JetBrains Mono Nerd Font)
- **Lowercase for all UI text** - headers, labels, buttons, everything
- Use **Nerd Fonts** for icons instead of SVG libraries
- Use `tabular-nums` for numbers that change (timers, counters)
- Simple, compact text - no unnecessary capitalization or styling

### Colors & Theme

- Minimal color palette with **semantic naming**
- Support both dark and light themes
- Use CSS variables for theming
- Prefer muted colors - let content and ASCII art shine
- Background textures use character patterns, not gradients

### Animation Principles

- Animations should be **smooth and purposeful**
- Use ASCII block characters (░▒▓█) for loading/shimmer effects
- Always support `animated={false}` for static variants
- Keep update intervals fast enough to feel smooth (100-150ms range)

## Component Patterns

### Interactive Elements

- Mark all interactive components as `"use client"`
- Use bracket notation for buttons: `[ button text ]`
- Add proper focus states (subtle, not distracting) - `focus:ring-1` not `focus:ring-2`
- Use smooth transitions on hover/focus
- **Keyboard-first navigation** - shortcuts for everything
- Use caret notation for shortcuts: `^k` not `⌘K` or `<kbd>`
- Non-interactive states should be visually distinct (opacity, cursor)

### Containers & Borders

- Use ASCII line-drawing characters for borders: `┌─┐│└─┘├┤┬┴┼`
- Support single and double borders (NOT rounded)
- Borders should be structural, not decorative
- Simple border styling - no fancy shadows or effects

### Backgrounds & Textures

- Use **Unicode block characters as backgrounds**: ░ (light), ▒ (medium), ▓ (dark), · (dot)
- Pattern should repeat at reduced opacity behind content
- Different patterns for different variants (default: ░, primary: ▓, destructive: ▒, outline: ·)
- Content layers on top with z-index

### Loading States

- Build **skeleton components** for locked/loading content
- Use animated gradients with ASCII block characters
- Loading states should feel dynamic, not static
- Maintain layout during loading (no content shifts)

### Lists & Collections

- Use simple left borders for list items
- Number items with zero-padding for alignment
- Differentiate states clearly (active: `◄`, disabled, locked)
- Subtle hover states with minimal transitions

### Keyboard Shortcuts

- Every major navigation action should have a keyboard shortcut
- Use `^` (caret) notation: `^k`, `^1`, `^i`, etc.
- Show shortcuts inline in menus (right-aligned, low opacity)
- Modal for keyboard shortcuts accessible via `^k`
- Use simple bracket notation: `[^k]` not fancy `<kbd>` elements
- Press `[esc]` to close, not "Press ESC"

## Layout Principles

### Structure

- **Simple, rigid layouts** - no fancy responsive grids
- Fixed sidebar width, flex main content
- Compact spacing - TUIs don't waste space
- Minimal padding - efficient use of screen real estate
- Header/sidebar/content/footer structure

### Text & Content

- **Constrained content width** for readability
- Consistent, compact spacing using Tailwind utilities
- Small text sizes (text-xs, text-sm) - TUIs are information-dense
- Line spacing is tight, not airy

### Responsive Design

- Mobile-first but don't compromise the TUI aesthetic
- Hide sidebar on mobile, show simplified menu
- Maintain monospace and ASCII character styling on all sizes

## Code Style

### General Principles

- **Lowercase everything** in UI - text, labels, buttons, headers
- Use simple string literals for brackets: `[x]`, `[ok]`, not components
- Keep components minimal - no over-engineering
- Focus states use `ring-1` not `ring-2` - subtle, not flashy

### Component Structure

- Use client components sparingly - only when needed
- Export both default and named exports for flexibility
- Props should be simple and intuitive
- Support `className` prop for customization

### Data Management

- Store content in JSON files for easy editing
- Use TypeScript interfaces for data structures
- Keep data separate from presentation logic
- Support metadata fields like `hidden`, `featured`, etc.

## Project Structure

### Single Source of Truth

- All components live in `registry/ascii/` - this is the only place components exist
- No duplication in `components/ascii/` - registry is the source
- Import components: `import Input from "@/registry/ascii/input"`
- The `ui` alias in `components.json` points to `@/registry/ascii`

### shadcn CLI Compatibility

- Registry definitions in root `registry.json`
- Each component has proper metadata and dependencies
- CSS variables defined for theming
- Types: `registry:ui`, `registry:component`, `registry:lib`
- Components installable via: `npx shadcn@latest add https://asciicn.fldr.zip/r/input.json`
- Manual copy from `registry/ascii/` is also supported

## Examples

### Good TUI Style

```tsx
// Button with brackets
<button>[ok]</button>
<button>[cancel]</button>

// Shortcuts with caret
press [^k] for shortcuts

// Headers
<h1>introduction</h1>
<h2>installation</h2>

// Close button
<button>[x]</button>
```

### Bad Style (Avoid)

```tsx
// Too fancy
<button className="rounded-full shadow-lg">OK</button>

// Wrong notation
<kbd>⌘K</kbd>
<kbd className="px-2 py-1 border">Ctrl+K</kbd>

// Capitalized
<h1>Introduction</h1>
<button>Cancel</button>
```

## Remember

This is a **TUI library**, not a modern web UI with ASCII decoration. Every design decision should ask: "Would this look at home in a terminal application?" If the answer is no, simplify it.

Use named exports for components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mergd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
