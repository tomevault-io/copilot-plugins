## claudeusagepage

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static landing page for "Usage for Claude", a macOS menu bar application that tracks Claude subscription usage. The page is built with plain HTML and CSS, with no build tools or JavaScript frameworks.

## Architecture

**Static Site Structure:**
- `index.html` - Single-page marketing site with hero, features, CTA, and FAQ sections
- `styles.css` - All styling with mobile-responsive breakpoints
- `Assets/` - Image assets for widgets and app screenshots

**Key Design Patterns:**

1. **Two-Column Feature Layout**: Feature sections use CSS Grid with `grid-template-columns: 1fr 1fr` for desktop, collapsing to single column on mobile (768px breakpoint)

2. **Absolute Positioned Menu View**: The menu screenshot (`.menu-column`) is absolutely positioned in the top-right of the tracking section on desktop, repositioned relatively on mobile

3. **Responsive Breakpoints**:
   - 1400px - Reduce padding, adjust menu view size
   - 1024px - Smaller typography, constrain widgets grid
   - 768px - Single column layout, reorder widgets section
   - 480px - Mobile-optimized sizing

4. **Brand Colors**: Primary brand color is `#c15f3c` (used for badges, stars, borders)

## Development

This is a static site with no build process. Simply open `index.html` in a browser to preview changes.

**Viewing the site:**
```bash
open index.html
```

Or use a local server:
```bash
python3 -m http.server 8000
# Then visit http://localhost:8000
```

## CSS Architecture Notes

- All animations use `cubic-bezier(0.16, 1, 0.3, 1)` easing for Apple-like polish
- Star decorations are SVG elements positioned absolutely with low opacity
- Drop shadows use `filter: drop-shadow()` for images to preserve transparency
- Typography uses SF Pro Display/Text with system font fallbacks

---
> Source: [hayek/ClaudeUsagePage](https://github.com/hayek/ClaudeUsagePage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
