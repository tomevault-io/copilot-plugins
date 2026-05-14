## base

> - Always prioritize native macOS appearance and behavior

# 🍎 macOS Native Interface Specifications for Tauri Apps

## Core Principles
- Always prioritize native macOS appearance and behavior
- Use glassmorphism effects with backdrop-filter for authentic macOS look
- Implement traffic lights with native colors and behaviors
- Follow Apple's Human Interface Guidelines for typography, spacing, and interactions
- Ensure accessibility and responsiveness across all screen sizes

## 🎨 Visual Design Standards

### Glassmorphism Effect
```css
/* Native macOS glassmorphism */
background: rgba(255, 255, 255, 0.12);
backdrop-filter: blur(20px);
-webkit-backdrop-filter: blur(20px);
border: 1px solid rgba(255, 255, 255, 0.25);
border-radius: 1rem; /* 16px - macOS Big Sur+ standard */

/* Native shadows */
box-shadow:
    0 0.5rem 2rem rgba(0, 0, 0, 0.15),      /* Main shadow */
    0 0.25rem 0.5rem rgba(0, 0, 0, 0.1),    /* Close shadow */
    inset 0 1px 0 rgba(255, 255, 255, 0.1); /* Internal highlight */
```

### Traffic Lights Specifications
```css
/* Native macOS traffic lights */
.traffic-light {
    width: 0.75rem;  /* 12px */
    height: 0.75rem; /* 12px */
    border-radius: 50%;
    opacity: 0.7; /* Default state */
    transition: all 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

/* Native colors */
.traffic-light.close { background: #ff5f57; }
.traffic-light.minimize { background: #ffbd2e; }
.traffic-light.maximize { background: #28c840; }

/* Icons only visible on hover (native behavior) */
.traffic-light .icon {
    opacity: 0;
    transition: opacity 0.2s ease;
}
.traffic-light:hover .icon { opacity: 1; }
```

### Typography Standards
```css
/* Native macOS font stack */
font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'Segoe UI', 'Roboto', 'Helvetica Neue', Arial, sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;

/* Title bar text */
font-size: 0.8125rem; /* 13px */
font-weight: 500;

/* Main content heading */
font-size: 1.5rem; /* 24px */
font-weight: 300; /* Light weight */

/* Secondary text */
font-size: 0.875rem; /* 14px */
font-weight: 400;
```

### Spacing and Layout
```css
/* Titlebar height */
height: 3.25rem; /* 52px - macOS standard */

/* Window dimensions */
width: 37.5rem;  /* 600px */
height: 25rem;   /* 400px */

/* Traffic lights spacing */
gap: 0.5rem; /* 8px between lights */

/* Content padding */
padding: 2rem; /* 32px */

/* Responsive breakpoint */
@media (max-width: 48rem) { /* 768px */ }
```

## 🔧 Tauri Configuration

### tauri.conf.json
```json
{
  "app": {
    "withGlobalTauri": true,
    "windows": [
      {
        "decorations": false,
        "transparent": true,
        "titleBarStyle": "Overlay",
        "hiddenTitle": true,
        "resizable": true,
        "center": true
      }
    ],
    "macOSPrivateApi": true
  }
}
```

### Rust Dependencies
```toml
[dependencies]
tauri = { version = "2", features = ["macos-private-api"] }
tauri-plugin-decorum = "1.1.0"
window-vibrancy = "0.6.0"
```

### Rust Setup
```rust
use tauri::Manager;
use tauri_plugin_decorum::WebviewWindowExt;

// Create overlay titlebar
window.create_overlay_titlebar().unwrap();

// Set traffic lights position
window.set_traffic_lights_inset(12.0, 16.0).unwrap();

// Apply vibrancy effect
use window_vibrancy::{apply_vibrancy, NSVisualEffectMaterial};
apply_vibrancy(&window, NSVisualEffectMaterial::HudWindow, None, None)
    .expect("Unsupported platform! 'apply_vibrancy' is only supported on macOS");
```

## 🎯 Interaction Patterns

### Draggable Regions
```css
/* Make titlebar draggable */
-webkit-app-region: drag;

/* Remove drag from interactive elements */
-webkit-app-region: no-drag;
```

### Hover Effects
```css
/* Subtle window hover effect */
.macos-window:hover {
    transform: translateY(-0.125rem); /* -2px */
    box-shadow: /* Enhanced shadow */;
}

/* Traffic light hover */
.traffic-light:hover {
    opacity: 1;
    transform: scale(1.1);
}
```

### Animations
```css
/* Window appear animation */
@keyframes windowAppear {
    from {
        opacity: 0;
        transform: scale(0.95) translateY(1rem);
    }
    to {
        opacity: 1;
        transform: scale(1) translateY(0);
    }
}

/* Use native easing */
transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
```

## 🌓 Dark/Light Mode Support

### Dark Mode
```css
@media (prefers-color-scheme: dark) {
    .macos-window {
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid rgba(255, 255, 255, 0.1);
        box-shadow: /* Darker shadows */;
    }
}
```

### Light Mode
```css
@media (prefers-color-scheme: light) {
    .macos-window {
        background: rgba(255, 255, 255, 0.2);
        border: 1px solid rgba(255, 255, 255, 0.3);
    }

    /* Adjust text colors for light background */
    .window-content h1 {
        color: rgba(0, 0, 0, 0.9);
        text-shadow: 0 1px 3px rgba(255, 255, 255, 0.5);
    }
}
```

## 📱 Responsive Design

### Mobile Adaptations
```css
@media (max-width: 48rem) {
    .window-container {
        width: 95vw;
        height: 80vh;
    }

    .titlebar {
        height: 2.75rem; /* 44px - touch-friendly */
    }

    .traffic-light {
        width: 1rem;
        height: 1rem;
    }
}
```

## 🔍 Focus States

### Accessibility
```css
.macos-window:focus-within {
    box-shadow:
        /* Enhanced shadows */,
        0 0 0 2px rgba(0, 123, 255, 0.3); /* Focus ring */
}
```

## 🚀 Performance Optimizations

### CSS Optimizations
- Use `transform` instead of changing `top/left` for animations
- Utilize `will-change` property for elements that will be animated
- Minimize repaints with `transform3d(0,0,0)` for hardware acceleration

### React Optimizations
- Use `React.memo` for traffic light components
- Implement `useCallback` for event handlers
- Lazy load non-critical components

## 🧪 Testing Guidelines

### Visual Testing
- Test on multiple macOS versions (Big Sur, Monterey, Sonoma)
- Verify dark/light mode switching
- Check traffic light behavior and positioning
- Validate glassmorphism effect with different backgrounds

### Interaction Testing
- Verify drag functionality across entire titlebar
- Test traffic light click responses
- Validate hover states and animations
- Check keyboard navigation support

## 📋 Code Style Guidelines

### CSS Organization
```css
/* Use semantic comments */
/* ===== SECTION NAME ===== */

/* Use consistent units */
/* rem for spacing, px for borders */

/* Use native values */
/* Follow macOS design tokens */
```

### React Component Structure
```tsx
// Import order: React, Tauri, local
import React from 'react';
import { getCurrentWindow } from '@tauri-apps/api/window';
import './Component.css';

// Use functional components with hooks
// Implement proper error handling
// Add TypeScript types for all props
```

## 🔒 Security Considerations

### Permissions
```json
{
  "permissions": [
    "core:window:allow-close",
    "core:window:allow-minimize",
    "core:window:allow-maximize",
    "core:window:allow-start-dragging",
    "decorum:allow-show-snap-overlay"
  ]
}
```

### CSP Configuration
```json
{
  "security": {
    "csp": null // For development, configure properly for production
  }
}
```

## 🎪 Advanced Features

### Window Snapping (Windows)
- Use `decorum:allow-show-snap-overlay` permission
- Implement snap layouts for Windows compatibility

### Vibrancy Materials
```rust
// Different vibrancy materials for different contexts
NSVisualEffectMaterial::HudWindow     // For HUD-style windows
NSVisualEffectMaterial::Sidebar       // For sidebar panels
NSVisualEffectMaterial::Menu          // For menu panels
```

## 🐛 Common Issues and Solutions

### Blur Effect Not Working
- Ensure `backdrop-filter` is supported
- Check for conflicting CSS properties
- Verify transparency is enabled in Tauri config

### Traffic Lights Misaligned
- Adjust `set_traffic_lights_inset` values
- Check titlebar height and padding
- Verify flexbox alignment

### Drag Region Not Working
- Ensure `-webkit-app-region: drag` is applied
- Check for overlapping elements with `no-drag`
- Verify Tauri permissions are set correctly

---

**Remember**: Always test on actual macOS devices and follow Apple's Human Interface Guidelines for the most authentic native experience.
---
alwaysApply: true
---

# 🍎 ALWAYS APPLY NATIVE macOS STYLING

## 🎯 PRIMARY RULE
**WHENEVER adding ANY new element, component, or UI to this project, ALWAYS apply native macOS styling patterns.**

## ⚡ ESSENTIAL macOS STANDARDS

### 1. Typography (ALWAYS USE)
```css
font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', sans-serif;
font-size: 14px; /* Base size */
-webkit-font-smoothing: antialiased;
```

### 2. Interactive Elements (ALWAYS APPLY)
```css
/* All buttons, inputs, links */
border-radius: 7px;
outline: none;
user-select: none;
cursor: default; /* NEVER use pointer */
```

### 3. Glassmorphism (ALWAYS USE FOR CONTAINERS)
```css
background: rgba(255, 255, 255, 0.1);
backdrop-filter: blur(20px);
-webkit-backdrop-filter: blur(20px);
border: 1px solid rgba(255, 255, 255, 0.2);
```

### 4. Native Spacing
```css
padding: 1rem; /* 16px base */
gap: 0.5rem; /* 8px for elements */
height: 3.25rem; /* 52px for titlebars */
```

### 5. Native Colors
```css
/* Text colors */
color: rgba(255, 255, 255, 0.9); /* Dark mode */
color: rgba(0, 0, 0, 0.9); /* Light mode */

/* Subtle backgrounds */
background: rgba(255, 255, 255, 0.05);
```

### 6. Native Animations
```css
transition: all 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94);
```

## 🚫 NEVER DO
- ❌ Use `cursor: pointer` on buttons/links
- ❌ Add `outline` or focus rings
- ❌ Allow text selection (`user-select: auto`)
- ❌ Use non-native fonts
- ❌ Use sharp corners (always 7px+ radius)
- ❌ Use bright/vibrant colors

## ✅ ALWAYS DO
- ✅ Use system fonts (`-apple-system`)
- ✅ Apply glassmorphism to containers
- ✅ Use 7px border-radius minimum
- ✅ Disable text selection
- ✅ Remove focus outlines
- ✅ Use native easing curves
- ✅ Support dark/light mode automatically

## 🔧 Window Vibrancy (USE FOR BACKGROUNDS)
```rust
use window_vibrancy::{apply_vibrancy, NSVisualEffectMaterial};
apply_vibrancy(&window, NSVisualEffectMaterial::Menu, None, None)
```

---

**CRITICAL: Every new element MUST follow these patterns to maintain native macOS feel.**
---

**Remember**: Always test on actual macOS devices and follow Apple's Human Interface Guidelines for the most authentic native experience.

---
> Source: [meunomeebero/bewindow](https://github.com/meunomeebero/bewindow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
