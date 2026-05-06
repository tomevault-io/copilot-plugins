## styling

> @import "tailwindcss";

# Styling Guidelines - Tailwind CSS v4

## Configuration

### Tailwind v4 Import Syntax
```css
/* globals.css */
@import "tailwindcss";
```

### Theme Variables (CSS Custom Properties)
Defined in `globals.css` for Mira theme variant:
```css
:root {
  --background: ...;
  --foreground: ...;
  --primary: ...;
  --secondary: ...;
  --muted: ...;
  --accent: ...;
  --destructive: ...;
  --card: ...;
  --popover: ...;
  --border: ...;
  --ring: ...;
}
```

## Color Usage

### Semantic Colors (Use These)
```tsx
// Backgrounds
className="bg-background"      // Page background
className="bg-card"            // Card backgrounds
className="bg-muted"           // Subtle backgrounds
className="bg-primary"         // Primary actions
className="bg-destructive"     // Danger/delete

// Text
className="text-foreground"    // Primary text
className="text-muted-foreground"  // Secondary text
className="text-primary"       // Accent text
className="text-destructive"   // Error text

// Borders
className="border-border"      // Default borders
className="border-primary"     // Accent borders
```

### Status Colors
```tsx
// Success (green)
className="text-green-600 dark:text-green-400"
className="bg-green-50 dark:bg-green-950"

// Warning (yellow/amber)
className="text-amber-600 dark:text-amber-400"
className="bg-amber-50 dark:bg-amber-950"

// Error (red)
className="text-red-600 dark:text-red-400"
className="bg-red-50 dark:bg-red-950"

// Info (blue)
className="text-blue-600 dark:text-blue-400"
className="bg-blue-50 dark:bg-blue-950"
```

## Custom Utility Classes (globals.css)

### Status Badges
```tsx
<span className="status-badge status-active">Active</span>
<span className="status-badge status-pending">Pending</span>
<span className="status-badge status-completed">Complete</span>
<span className="status-badge status-draft">Draft</span>
```

### Stat Cards
```tsx
<div className="stat-card">
  <span className="stat-label">Total Budget</span>
  <span className="stat-value">$125,000</span>
</div>

<div className="stat-card-compact">
  <span className="stat-label">Items</span>
  <span className="stat-value">24</span>
</div>
```

### Table Styling
```tsx
<thead className="table-header">
<tr className="table-row-hover">
<td className="col-underwriting">$50,000</td>
<td className="col-forecast">$52,000</td>
<td className="col-actual">$48,500</td>
```

### Form Inputs
```tsx
<input className="form-input" />
<select className="form-select" />
<input className="inline-input" />  // For inline editing
```

### Empty States
```tsx
<div className="empty-state">
  <IconInbox className="empty-state-icon" />
  <p>No items yet</p>
</div>

<div className="empty-state-lg">
  <IconFolderOpen className="empty-state-icon" />
  <h3>No projects</h3>
  <p>Create your first project to get started</p>
</div>
```

### Section Headers
```tsx
<h2 className="section-header">Budget Details</h2>
<h3 className="section-subheader">Kitchen Items</h3>
<h1 className="section-header-lg">Dashboard</h1>
```

### Icon Sizes
```tsx
<IconPlus className="icon-xs" />   // 14px
<IconPlus className="icon-sm" />   // 16px
<IconPlus className="icon-md" />   // 20px
<IconPlus className="icon-lg" />   // 24px
<IconPlus className="icon-xl" />   // 32px
```

### Animations
```tsx
<div className="fade-in">...</div>
<div className="scale-in">...</div>
<div className="slide-in-bottom">...</div>
<div className="modal-enter">...</div>
<div className="dropdown-enter">...</div>
```

### Transitions
```tsx
<button className="transition-base hover:bg-muted">
<button className="transition-fast hover:opacity-80">
<div className="hover-lift">  // Subtle lift on hover
```

## Spacing Conventions

### Component Spacing
```tsx
// Card padding
className="p-4"        // Standard
className="p-6"        // Larger cards

// Section gaps
className="space-y-4"  // Between items
className="space-y-6"  // Between sections
className="gap-4"      // Grid/flex gaps
```

### Page Layout
```tsx
// Container
className="container mx-auto px-4 py-6"

// Max widths
className="max-w-4xl"  // Forms
className="max-w-6xl"  // Content
className="max-w-7xl"  // Full pages
```

## Typography

### Fonts (Fontsource Variable)
- **Sans**: Inter Variable
- **Mono**: JetBrains Mono Variable

### Text Sizes
```tsx
className="text-xs"    // 12px - Labels, badges
className="text-sm"    // 14px - Secondary text
className="text-base"  // 16px - Body text
className="text-lg"    // 18px - Subheadings
className="text-xl"    // 20px - Card titles
className="text-2xl"   // 24px - Section headers
className="text-3xl"   // 30px - Page titles
className="text-4xl"   // 36px - Hero metrics
```

### Font Weights
```tsx
className="font-normal"    // 400
className="font-medium"    // 500
className="font-semibold"  // 600
className="font-bold"      // 700
```

## Dark Mode

### Theme Provider
Uses `next-themes` with `data-theme` attribute.

### Dark Mode Classes
```tsx
// Automatic switching
className="bg-white dark:bg-gray-900"
className="text-gray-900 dark:text-gray-100"

// Use semantic colors (preferred)
className="bg-background text-foreground"  // Auto-switches
```

## Responsive Design

### Breakpoint Prefixes
```tsx
// Mobile-first approach
className="w-full md:w-1/2 lg:w-1/3"
className="flex flex-col md:flex-row"
className="hidden lg:block"
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4"
```

### Common Responsive Patterns
```tsx
// Stack → Row
<div className="flex flex-col md:flex-row gap-4">

// Full → Half → Quarter
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">

// Hide on mobile
<div className="hidden md:flex">

// Show only on mobile
<div className="md:hidden">
```

## Animation with Framer Motion

```tsx
import { motion } from 'framer-motion';

// Fade in
<motion.div
  initial={{ opacity: 0 }}
  animate={{ opacity: 1 }}
  transition={{ duration: 0.2 }}
>

// Slide up
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
>

// Stagger children
<motion.div
  initial="hidden"
  animate="visible"
  variants={{
    visible: { transition: { staggerChildren: 0.05 } }
  }}
>
```

---
> Source: [adamj-ops/rehab-budget-pro](https://github.com/adamj-ops/rehab-budget-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
