## openmat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenMat is a React single-page application showcasing smooth scrolling and animation effects. It uses Locomotive Scroll for parallax scrolling and GSAP for entrance animations.

## Commands

```bash
npm run dev      # Start development server (Vite HMR)
npm run build    # Production build to dist/
npm run lint     # Run ESLint
npm run preview  # Preview production build
```

## Architecture

### Tech Stack
- React 19 with Vite 7
- Locomotive Scroll 4.1 for smooth scrolling/parallax
- GSAP for animations
- ESLint (flat config, v9+)
- Reactbits for animated components

### Component Pattern
All section components follow this pattern:
- Wrap content in `data-scroll-section` for Locomotive Scroll
- Use `data-scroll` and `data-scroll-speed` for parallax effects
- CSS class names match component names (lowercase)

### Scroll System
- `App.jsx` initializes Locomotive Scroll on mount with `smooth: true` and `lerp: 0.08`
- The scroll container is `<main data-scroll-container>`
- Call `locomotiveScrollRef.current?.update()` after DOM changes that affect layout
- Components use `is-inview` class for scroll-triggered visibility

### GSAP Animations
Entry animations are defined in `App.jsx` using `gsap.fromTo()` targeting CSS classes like `.hero-card`, `.hero-tagline`, `.nav`.

### Component Structure
```
src/
├── App.jsx          # Scroll/animation init, composes sections
├── App.css          # Global styles, Locomotive Scroll base
├── components/      # Section components (Nav, Hero, Problem, Services, Intel, Founder, Contact)
└── main.jsx         # React entry point
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShuhaiYu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ShuhaiYu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
