## terragon-oss

> Terragon theme colors and design system guidelines

# Terragon Theme Colors Usage Guidelines

## Overview
Always use the semantic theme colors defined in [apps/www/src/app/globals.css](mdc:apps/www/src/app/globals.css) instead of hardcoded Tailwind color classes. This ensures consistent theming and proper dark mode support across the Terragon AI coding assistant platform.

## Theme Color System

### Primary Color Variables
- `--primary: #2e7d32` (forest green) / `--primary: #4caf50` (dark mode)
- `--primary-foreground: #ffffff` / `--primary-foreground: #0a1f0c` (dark mode)
- `--secondary: #e8f5e9` (light green) / `--secondary: #3e4a3d` (dark mode)
- `--secondary-foreground: #1b5e20` / `--secondary-foreground: #d7e0d6` (dark mode)
- `--accent: #c8e6c9` (pale green) / `--accent: #388e3c` (dark mode)
- `--accent-foreground: #1b5e20` / `--accent-foreground: #f0ebe5` (dark mode)
- `--destructive: #c62828` (red) / `--destructive: #c62828` (dark mode)
- `--destructive-foreground: #ffffff` / `--destructive-foreground: #f0ebe5` (dark mode)

### Layout & Background Colors
- `--background: #f8f5f0` (warm off-white) / `--background: #1c2a1f` (dark green)
- `--foreground: #3e2723` (dark brown) / `--foreground: #f0ebe5` (warm white)
- `--card: #f8f5f0` (matches background) / `--card: #2d3a2e` (dark mode)
- `--card-foreground: #3e2723` / `--card-foreground: #f0ebe5` (dark mode)
- `--popover: #f8f5f0` / `--popover: #2d3a2e` (dark mode)
- `--popover-foreground: #3e2723` / `--popover-foreground: #f0ebe5` (dark mode)
- `--muted: #f0e9e0` (subtle beige) / `--muted: #2d3a2e` (dark mode)
- `--muted-foreground: #6d4c41` (muted brown) / `--muted-foreground: #d7cfc4` (dark mode)

### Interaction & Border Colors
- `--border: #e0d6c9` (warm border) / `--border: #3e4a3d` (dark mode)
- `--input: #e0d6c9` (matches border) / `--input: #3e4a3d` (dark mode)
- `--ring: #2e7d32` (focus ring) / `--ring: #4caf50` (dark mode)

### Sidebar Colors
- `--sidebar: #f0e9e0` / `--sidebar: #1c2a1f` (dark mode)
- `--sidebar-foreground: #3e2723` / `--sidebar-foreground: #f0ebe5` (dark mode)
- `--sidebar-primary: #2e7d32` / `--sidebar-primary: #4caf50` (dark mode)
- `--sidebar-primary-foreground: #ffffff` / `--sidebar-primary-foreground: #0a1f0c` (dark mode)
- `--sidebar-accent: #c8e6c9` / `--sidebar-accent: #388e3c` (dark mode)
- `--sidebar-accent-foreground: #1b5e20` / `--sidebar-accent-foreground: #f0ebe5` (dark mode)
- `--sidebar-border: #e0d6c9` / `--sidebar-border: #3e4a3d` (dark mode)
- `--sidebar-ring: #2e7d32` / `--sidebar-ring: #4caf50` (dark mode)

### Chart Colors (for data visualization)
- `--chart-1: #4caf50` / `--chart-1: #81c784` (dark mode)
- `--chart-2: #388e3c` / `--chart-2: #66bb6a` (dark mode)
- `--chart-3: #2e7d32` / `--chart-3: #4caf50` (dark mode)
- `--chart-4: #1b5e20` / `--chart-4: #43a047` (dark mode)
- `--chart-5: #0a1f0c` / `--chart-5: #388e3c` (dark mode)

## Typography & Design System

### Fonts
- `--font-sans: Montserrat, sans-serif` - Primary UI font
- `--font-serif: Merriweather, serif` - Editorial content
- `--font-mono: Source Code Pro, monospace` - Code and technical content

### Border Radius
- `--radius: 0.5rem` - Base radius (8px)
- `--radius-sm: calc(var(--radius) - 4px)` - Small radius (4px)
- `--radius-md: calc(var(--radius) - 2px)` - Medium radius (6px)
- `--radius-lg: var(--radius)` - Large radius (8px)
- `--radius-xl: calc(var(--radius) + 4px)` - Extra large radius (12px)

## Usage Rules

### ✅ DO Use Theme Colors
```tsx
// Text colors
className="text-foreground"           // Main text
className="text-muted-foreground"     // Secondary text
className="text-primary"              // Primary actions/highlights
className="text-destructive"          // Errors/warnings

// Background colors
className="bg-background"             // Main background
className="bg-card"                   // Card backgrounds
className="bg-muted"                  // Muted backgrounds
className="bg-primary/10"             // Primary with opacity
className="bg-destructive/10"         // Error backgrounds with opacity

// Border colors
className="border-border"             // Default borders
className="border-primary/20"         // Primary borders with opacity
className="border-destructive/20"     // Error borders with opacity

// Sidebar colors
className="bg-sidebar"                // Sidebar background
className="text-sidebar-foreground"   // Sidebar text
className="bg-sidebar-accent/10"      // Sidebar accents
```

### ❌ DON'T Use Hardcoded Colors
```tsx
// Avoid these hardcoded color classes
className="text-gray-500"             // Use text-muted-foreground instead
className="bg-blue-100"               // Use bg-primary/10 instead
className="border-red-200"            // Use border-destructive/20 instead
className="text-green-600"            // Use text-primary instead
className="bg-green-50"               // Use bg-accent/10 instead
```

## Component Examples

### Status Indicators (for threads, sandboxes, etc.)
```tsx
// Good: Using theme colors for status badges
const getStatusStyles = (status: string) => {
  switch (status) {
    case "completed":
    case "running":
      return "bg-primary/10 text-primary border border-primary/20";
    case "error":
    case "failed":
      return "bg-destructive/10 text-destructive border border-destructive/20";
    case "pending":
    case "hibernating":
    default:
      return "bg-accent/10 text-accent-foreground border border-accent/20";
  }
};
```

### Interactive Elements (chat UI, buttons)
```tsx
// Good: Theme-aware hover states
className="text-muted-foreground hover:text-foreground border-border hover:border-accent transition-colors"

// Good: Primary action button
className="bg-primary text-primary-foreground hover:bg-primary/90"

// Good: Secondary action button
className="bg-secondary text-secondary-foreground hover:bg-secondary/80"
```

### Sidebar Navigation
```tsx
// Good: Sidebar item styling
className="text-sidebar-foreground hover:text-sidebar-primary hover:bg-sidebar-accent/20"

// Good: Active sidebar item
className="bg-sidebar-primary text-sidebar-primary-foreground"
```

## Terragon-Specific Patterns

### Chat Message Status
```tsx
// Claude response message
className="bg-card border border-border text-card-foreground"

// User message
className="bg-primary/5 border border-primary/20 text-foreground"

// System/tool message
className="bg-muted border border-muted text-muted-foreground"
```

### Code & Terminal UI
```tsx
// Terminal/bash output
className="bg-muted text-muted-foreground font-mono border border-border"

// Code blocks
className="bg-card text-card-foreground font-mono border border-border"

// Syntax highlighting should use chart colors for variety
```

### AI Coding Assistant Features
```tsx
// Sandbox status indicators
className="bg-primary/10 text-primary"      // Active sandbox
className="bg-muted text-muted-foreground"  // Hibernating sandbox
className="bg-destructive/10 text-destructive" // Error state

// GitHub integration status
className="bg-accent/10 text-accent-foreground" // PR created
className="bg-primary/10 text-primary"          // Commit successful
```

## File References
- Theme definitions: [apps/www/src/app/globals.css](mdc:apps/www/src/app/globals.css)
- Chat components: [apps/www/src/components/chat/](mdc:apps/www/src/components/chat)
- Status components: [apps/www/src/components/pr-status-pill.tsx](mdc:apps/www/src/components/pr-status-pill.tsx)

## Benefits
1. **Automatic dark mode support** - Colors adapt based on theme
2. **Consistent visual hierarchy** - Semantic naming ensures proper contrast
3. **Easy theme updates** - Change colors in one place affects entire app
4. **Better accessibility** - Theme colors are designed with proper contrast ratios
5. **Future-proof** - New themes can be added without touching components
6. **Platform cohesion** - Maintains Terragon's natural, coding-focused aesthetic

## Migration Checklist
When updating components:
- [ ] Replace `text-gray-*` with `text-muted-foreground` or `text-foreground`
- [ ] Replace `bg-gray-*` with `bg-muted` or `bg-card`
- [ ] Replace `border-gray-*` with `border-border`
- [ ] Replace color-specific classes (`text-blue-*`, `bg-green-*`) with semantic equivalents
- [ ] Use opacity modifiers (`/10`, `/20`) for subtle backgrounds and borders
- [ ] Test in both light and dark modes
- [ ] Ensure proper contrast for accessibility
- [ ] Use appropriate sidebar colors for navigation elements
- [ ] Apply consistent chart colors for data visualization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/terragon-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
