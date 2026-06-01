## board-facing-careerlaunch101

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
High-converting sponsorship landing page for Ontario's largest virtual career fair (December 2, 2025). Single-page Next.js application targeting corporate sponsors to book consultations.

## Common Development Commands
```bash
# Development
npm run dev          # Start dev server with Turbo on port 3000
npm run dev:host     # Dev server accessible from network
npm run dev:port     # Dev server on alternate port 3001

# Production
npm run build        # Create production build
npm run start        # Run production server

# Utilities
npm run lint         # Run Next.js linter
npm run clean        # Clear cache and .next folder
npm run restart      # Kill and restart dev server
npm run check        # Check if server is running
```

## Architecture Overview

### Tech Stack
- **Next.js 15.4.5** with App Router and TypeScript
- **Tailwind CSS** with comprehensive design system
- **React 19.1.1** with hooks for state management
- **clsx + tailwind-merge** for dynamic styling
- **Lucide React** for consistent iconography

### Key Design Patterns
1. **Component Composition**: All components use TypeScript interfaces with the `cn()` utility for class merging
2. **Mobile-First**: Every component starts at 320px width with progressive enhancement
3. **Performance First**: Intersection Observer for lazy loading, CSS animations over JavaScript
4. **Accessibility Built-in**: All interactive elements have keyboard support and ARIA labels

### Component Architecture
```typescript
// All components follow this pattern:
interface ComponentProps extends React.HTMLAttributes<HTMLElement> {
  variant?: 'primary' | 'secondary'
  // Additional props
}

export const Component: React.FC<ComponentProps> = ({ variant, className, ...props }) => {
  return (
    <element className={cn(baseStyles, variants[variant], className)} {...props}>
      {/* Content */}
    </element>
  )
}
```

### Section Component Pattern
All section components (Hero, About, Tiers, FAQ) follow this structure:
- Located in `src/components/sections/`
- Use `'use client'` directive for interactivity
- Implement scroll-triggered animations with Intersection Observer
- Include TypeScript interfaces extending `React.HTMLAttributes<HTMLElement>`
- Support className override via `cn()` utility

### State Management Strategy
- **Local State**: React hooks for component state
- **Navigation State**: Intersection Observer tracks active sections
- **Animation State**: CSS-based with JavaScript triggers
- **No Redux/Context**: Single-page app doesn't require global state

### Design System Integration
The design system is implemented through a dual approach:
1. **Tailwind Config**: Extended with project colors, typography, spacing, animations
2. **CSS Custom Properties**: Defined in globals.css for runtime theming and consistency
3. **Utility Classes**: Custom utilities for skeleton loading, focus states, accessibility helpers

### Critical Performance Optimizations
1. **Logo Carousel**: CSS animation with 35s duration, pauses on hover
2. **Statistics Counters**: Only animate when visible via Intersection Observer
3. **Navigation**: Uses `position: sticky` with backdrop blur for performance
4. **Images**: All images must use Next.js Image component with lazy loading
5. **Intersection Observer Pattern**: Used consistently across all sections for animation triggers

## Key Implementation Details

### Navigation Behavior
- Sticky positioning with 80px height (64px mobile)
- Intersection Observer with `-20% 0px -70% 0px` rootMargin for section tracking
- Smooth scroll with offset calculation for sticky header
- Mobile menu prevents body scroll when open
- Dual implementation (desktop/mobile) for optimal UX

### Component Data Patterns
```typescript
// Tier data structure in src/data/tiers.ts
interface TierConfig {
  id: string
  name: string
  price: number
  availability: number | 'unlimited'
  limited?: boolean
  featured?: boolean
  benefits: string[]
}

// FAQ data structure in src/data/faq.ts
interface FAQItem {
  id: string
  question: string
  answer: string
  category?: 'event-details' | 'sponsorship-process'
  keywords?: string[]
}
```

### Animation Architecture
- **CSS Keyframes**: Defined in Tailwind config for optimal performance
- **Intersection Observer**: Triggers animations only when elements are visible
- **Staggered Delays**: Components use incremental delays for smooth sequences
- **Reduced Motion**: Comprehensive support via CSS media queries
- **Timing Standards**: Fast (150ms), Standard (250ms), Slow (400ms)

### Bento Grid System
Recent addition for flexible card layouts:
```typescript
// BentoGrid component provides responsive grid container
// BentoCard component with hover effects and optional backgrounds
// Usage pattern: col-span-3 desktop:col-span-1/2 for responsive layouts
```

## Core Requirements

### Performance Targets
- LCP < 2.5 seconds on mobile networks
- Lighthouse score 90+ across all categories
- Touch targets minimum 48px
- All content lazy-loaded below fold

### Accessibility Standards
- WCAG 2.1 AA compliance mandatory
- Color contrast: 4.5:1 (normal text), 3:1 (large text)
- Keyboard navigation for all interactive elements
- Screen reader compatible with semantic HTML
- Focus-visible styles with 3px blue outline

### Mobile-First Breakpoints
- Mobile: 320px (default)
- Tablet: 768px
- Desktop: 1024px
- Large: 1200px

## Project Status
**Current State**: All major sections complete - Navigation, Hero, LogoCarousel, About (with bento grid), Tiers, and FAQ sections implemented

**Business Goal**: Convert corporate visitors into consultation bookings through urgency and clear value proposition.

## Custom Sub-Agents

This project includes custom Claude Code sub-agents in `.claude/agents/`:
- **@senior-frontend-engineer** - Systematic frontend implementation specialist for building production-ready components following established architectural patterns

Use `@agent-name` to invoke a sub-agent, or `/agents` to see all available agents.

## Development Patterns

### Component Creation Workflow
1. Create component in appropriate directory (`ui/`, `sections/`, `layout/`)
2. Follow TypeScript interface pattern extending HTML attributes
3. Implement responsive design mobile-first
4. Add Intersection Observer for animations if needed
5. Use `cn()` utility for all className composition
6. Test at minimum 320px width

### Data-Driven Components
- All dynamic content should be configuration-driven
- Use TypeScript interfaces for type safety
- Centralize data in `src/data/` directory
- Components should render from configuration objects

### Animation Implementation
1. Use CSS animations via Tailwind classes when possible
2. Implement Intersection Observer for scroll-triggered animations
3. Always include `prefers-reduced-motion` fallbacks
4. Stagger animation delays for sequences (100ms increments)

## Common Issues & Solutions

### CSS Class Conflicts
The project uses standard Tailwind color classes (e.g., `text-black`, `text-gray-700`). Always check Tailwind config for custom extensions before using classes.

### Missing Animations
Some animations may be referenced but not defined in Tailwind config. Either remove animation classes or add them to the config keyframes section.

### Container Usage
Use the `.container` utility class for consistent max-width and responsive padding instead of custom implementations.

### Focus States
All interactive elements automatically receive focus-visible styles via globals.css. Don't override unless necessary.

### Sticky Table Headers
Comparison tables in the Tiers section use sticky `<thead>` elements. Desktop tables position at `top-[80px]` and mobile at `top-[64px]` to account for navigation height.

## Development Workflow
1. Check if server is running with `npm run check`
2. Use `npm run dev` for development with Turbo compilation
3. Test mobile-first at 320px width
4. Verify animations respect reduced-motion preferences  
5. Ensure all touch targets meet 48px minimum
6. Run `npm run lint` before committing changes

## Configuration Notes

### TypeScript Configuration
- Strict mode enabled
- Path alias `@/*` maps to `./src/*`
- Target ES2017 for compatibility

### Next.js Configuration
- Images allowed from `i.imgur.com` domain
- App Router enabled
- Turbo development server for faster builds

### No Test Framework
Currently no test framework is configured. Components are tested manually during development.

---
> Source: [damianstory/board_facing_careerlaunch101](https://github.com/damianstory/board_facing_careerlaunch101) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
