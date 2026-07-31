## tailwindcss-v4

> description: Tailwindcss 4 rules


---
description: Tailwindcss 4 rules
---
# Tailwind CSS 4 Rules

## Overview

Tailwind CSS 4 introduces a major shift towards CSS-first configuration, modern web features, and improved performance. This guide covers the key changes and best practices getting rid of old standards like tailwind.config.js

## Installation & Setup

### Basic Setup
```css
@import "tailwindcss";
```

### CSS-First Configuration
```css
@import "tailwindcss";

@theme {
  --font-family-display: "Satoshi", "sans-serif";
  --breakpoint-3xl: 1920px;
  --color-neon-pink: oklch(71.7% 0.25 360);
}
```

### Custom Theme Setup
```css
@import "tailwindcss";
@import "tailwindcss/preflight" layer(base);
@import "tailwindcss/utilities" layer(utilities);

@theme {
  --color-*: initial;
  --color-gray-50: #f8fafc;
  --color-gray-100: #f1f5f9;
  /* ... */
}
```

## Modern CSS Features

### Cascade Layers
```css
@layer theme, base, components, utilities;

@layer utilities {
  .mx-6 {
    margin-inline: calc(var(--spacing) * 6);
  }
}
```

### Registered Custom Properties
```css
@property --tw-gradient-from {
  syntax: "<color>";
  inherits: false;
  initial-value: #0000;
}
```

### Color Mix
```css
.bg-blue-500\/50 {
  background-color: color-mix(in oklab, var(--color-blue-500) 50%, transparent);
}
```

## Container Queries

### Basic Container Query
```html
<div class="@container">
  <div class="grid grid-cols-1 @sm:grid-cols-3 @lg:grid-cols-4">
    <!-- ... -->
  </div>
</div>
```

### Max-Width Container Query
```html
<div class="@container">
  <div class="grid grid-cols-3 @max-md:grid-cols-1">
    <!-- ... -->
  </div>
</div>
```

### Container Query Ranges
```html
<div class="@container">
  <div class="flex @min-md:@max-xl:hidden">
    <!-- ... -->
  </div>
</div>
```

## Dynamic Utilities

### Grid Columns
```html
<div class="grid grid-cols-15">
  <!-- ... -->
</div>
```

### Data Attributes
```html
<div data-current class="opacity-75 data-current:opacity-100">
  <!-- ... -->
</div>
```

### Dynamic Spacing
```css
@layer theme {
  :root {
    --spacing: 0.25rem;
  }
}

@layer utilities {
  .mt-8 {
    margin-top: calc(var(--spacing) * 8);
  }
  .w-17 {
    width: calc(var(--spacing) * 17);
  }
}
```

## Breaking Changes

### Removed Features
- Deprecated utilities (text-opacity-*, flex-grow-*, decoration-slice)
- PostCSS plugin and CLI in main package
- Default border color (now defaults to currentColor)
- Default ring width (now 1px instead of 3px)

### New Features
- P3 color palette with oklch
- 3D transform utilities
- @starting-style for transitions
- not-* variant
- color-scheme support
- field-sizing
- complex shadows
- inert support

## Performance Optimizations

### Build Performance
- Full builds: 5x faster
- Incremental builds: 100x faster
- Measured in microseconds

### Modern Web Optimizations
- Native cascade layers
- Registered custom properties
- color-mix() support
- Logical properties for RTL

## Theme Variables

### Accessing Theme Values
```html
<div class="p-[calc(var(--spacing-6)-1px)]">
  <!-- ... -->
</div>
```

### UI Library Integration
```jsx
import { motion } from "framer-motion"
// Theme values are available as CSS variables
```

## Best Practices

1. Use CSS-first configuration over JavaScript config
2. Leverage modern CSS features for better performance
3. Use container queries for component-based responsive design
4. Take advantage of dynamic utilities for flexible layouts
5. Use P3 color palette for more vivid colors
6. Implement proper cascade layers for better CSS organization
7. Use logical properties for better RTL support
8. Leverage CSS variables for theme customization

---
> Source: [fivelity/usmp](https://github.com/fivelity/usmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
