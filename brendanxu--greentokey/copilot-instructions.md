## greentokey

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GreenLink Capital is a professional enterprise website focused on tokenization of China's CCER (Certified Emission Reduction) green assets. This is a Next.js application with a modular architecture designed for RWA (Real World Assets) tokenization in green finance.

**Key Design Philosophy**: Complete UI/UX replication of ADDX.co design system adapted for green finance theme.

## Common Development Commands

```bash
# Development
npm run dev              # Start development server (localhost:3000)
npm run build           # Production build - ALWAYS run before commits
npm run start           # Start production server
npm run lint            # ESLint checking - run after major changes

# Install dependencies
npm install             # Install all dependencies
```

**Important**: Always run `npm run build` before committing to ensure TypeScript compilation succeeds.

## Architecture Overview

### Project Structure
This is a **Next.js Pages Router** application with a highly modular component architecture:

```
greentokey-website/
├── pages/                    # Next.js Pages Router
│   ├── _app.tsx             # Global app configuration  
│   ├── _document.tsx        # HTML document structure
│   └── index.tsx            # Main homepage layout
├── components/               # Modular component system
│   ├── ui/                  # Atomic components (Button, Card)
│   ├── layout/              # Layout components (Header)
│   ├── compound/            # Business logic components
│   └── sections/            # Page section components  
├── data/                    # Data and content management
│   ├── types/               # Complete TypeScript type system
│   ├── content/             # All text content (sections.ts)
│   └── config/              # Site configuration (site.ts)
├── lib/                     # Utilities and logic
│   ├── hooks/               # Custom React hooks
│   ├── utils/               # Utility functions
│   ├── constants/           # Design tokens and constants
│   └── validations/         # Form validation schemas
└── styles/                  # Styling system
    └── globals.css          # Global styles with ADDX design system
```

### Key Architectural Principles

**1. Data-Content Separation**: All text content is managed in `data/content/sections.ts` with complete TypeScript interfaces in `data/types/index.ts`. Never hardcode content in components.

**2. Atomic Component Design**: UI components in `components/ui/` use `class-variance-authority` for variant management. Each component has props interfaces and variant definitions.

**3. Custom Hooks Architecture**: Complex logic extracted into custom hooks in `lib/hooks/`:
- `useScroll`: Scroll state management with threshold detection
- `useMobileMenu`: Mobile menu state and interactions
- `useInView`: Intersection Observer wrapper for animations

**4. Type-Safe Development**: Comprehensive TypeScript types defined in `data/types/index.ts`. All components, content, and configurations are fully typed.

## Design System

### ADDX-Inspired Design System
The project replicates ADDX.co's design language with precision:

**Colors** (defined in `tailwind.config.js`):
- Primary Blue: `#0052FF` (ADDX signature color)
- Green Primary: `#00D4AA` (Green finance adaptation)
- Complete color palette for text, backgrounds, borders

**Typography**: 
- Font: Manrope (exact ADDX font)
- Complete typographic scale: h1-desktop (60px) → caption-mobile (12px)
- Precise line-heights, letter-spacing, font-weights

**Spacing**: 4px-based spacing system matching ADDX grid standards

**Animations**: Framer Motion integration with custom keyframes and transitions

### Component Variants

**Button Component**: 5 variants with full animation support:
- `primary`, `secondary`, `tertiary`, `green-primary`, `green-secondary`
- Loading states, icon support, size variants

**Card Component**: Feature cards and testimonial cards with hover animations

## Tech Stack & Dependencies

### Core Framework
- **Next.js 15.4.4** with Pages Router (not App Router)
- **TypeScript** with strict mode enabled
- **React 19.1.0** with modern hooks patterns

### Styling & Animation  
- **Tailwind CSS 3.4.17** with extensive custom configuration
- **Framer Motion 12.23.11** for animations and page transitions
- **Class Variance Authority** for component variant management

### UI & Icons
- **Lucide React** for icons (primary icon library)
- **Heroicons** for additional icons
- **Radix UI Icons** for specific UI elements

### Development Tools
- **ESLint** with Next.js configuration
- **PostCSS** with Autoprefixer
- **TypeScript 5.8.3** with strict settings

## Content Management

### Adding New Sections
1. Define TypeScript interface in `data/types/index.ts`
2. Add content data in `data/content/sections.ts`
3. Create section component in `components/sections/`
4. Import and use in `pages/index.tsx`

### Modifying Existing Content
- **All text content** lives in `data/content/sections.ts`
- **Site configuration** in `data/config/site.ts`
- **Never hardcode content** in component files

## Component Development

### Creating New UI Components
1. Use `components/ui/Button.tsx` as reference for structure
2. Define props interface with TypeScript
3. Use `cva` for variant management if multiple styles needed
4. Export from `components/ui/index.ts`
5. Add to type definitions if needed

### Animation Patterns
- Use Framer Motion's `motion` components
- Leverage predefined animations in `lib/utils/animations.ts`
- Follow ADDX animation timing and easing patterns

## Business Domain Context

### Green Finance Focus
This website serves institutional investors interested in:
- **CCER Tokenization**: China's Certified Emission Reduction credits
- **RWA (Real World Assets)**: Blockchain-based asset tokenization
- **ESG Compliance**: Environmental, Social, Governance investing
- **Institutional Services**: Professional-grade green finance solutions

### Target Audience
- Institutional investors and fund managers
- Asset owners with green/carbon assets
- Financial institutions seeking ESG exposure
- Compliance officers in green finance

### Regulatory Considerations
- **China Mainland**: Not targeted (per legal disclaimers)
- **Hong Kong SAR**: Primary jurisdiction, professional investors only
- **International**: Institutional clients with appropriate regulatory status

## Critical Implementation Notes

### Next.js Configuration
- Uses **Pages Router**, not App Router
- Images configured for Cloudinary and Unsplash domains
- ESLint and TypeScript errors fail builds (production-ready standards)

### Build Process
- **Production Build**: Must pass TypeScript compilation without errors
- **Static Generation**: Generates static pages for optimal performance
- **Bundle Analysis**: Monitor bundle size (currently ~140kB first load)

### Mobile Responsiveness
- **Mobile-first design** with progressive enhancement
- Complex navigation requires mobile menu implementation
- Touch-friendly interactions and proper viewport handling

### Performance Considerations
- Framer Motion animations optimized for 60fps
- Image optimization through Next.js Image component
- Code splitting through dynamic imports where needed

## Development Workflow

### Before Making Changes
1. Run `npm run dev` to start development server
2. Check current TypeScript errors with `npm run build`
3. Understand existing component patterns before creating new ones

### After Making Changes
1. Test responsive design across breakpoints
2. Verify animations work smoothly
3. Run `npm run lint` to check code style
4. **Always run `npm run build`** to ensure production readiness
5. Test accessibility with keyboard navigation

### Common Pitfalls
- Don't hardcode content - use data layer
- Don't break TypeScript compilation
- Maintain ADDX design system consistency
- Test mobile navigation thoroughly
- Verify Framer Motion performance

This codebase prioritizes maintainability, type safety, and design system consistency. Always follow the established patterns and architectural principles when making changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brendanxu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brendanxu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
