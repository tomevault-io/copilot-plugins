## portfolio3-0

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Start development server**: `pnpm dev` or `npm run dev`
- **Build for production**: `pnpm build` or `npm run build`
- **Start production server**: `pnpm start` or `npm run start`
- **Run linting**: `pnpm lint` or `npm run lint`

Note: This project uses pnpm as the preferred package manager (evident from pnpm-lock.yaml).

## Project Architecture

This is a Next.js 14 portfolio website with the following key characteristics:

### Framework & Technology Stack
- **Next.js 14** with App Router architecture
- **TypeScript** for type safety
- **Tailwind CSS v4** for styling with custom CSS variables
- **Shadcn/ui** components (configured in components.json)
- **Geist font** (Sans and Mono variants)
- **Vercel Analytics** integrated

### Directory Structure
```
├── app/                    # Next.js App Router
│   ├── globals.css        # Global styles with CSS variables
│   ├── layout.tsx         # Root layout component
│   ├── page.tsx           # Home page component
│   └── loading.tsx        # Loading UI
├── components/            # React components
│   └── theme-provider.tsx # next-themes provider
├── lib/
│   └── utils.ts          # Utility functions (cn helper)
├── styles/
│   └── globals.css       # Additional global styles
└── public/               # Static assets
```

### Styling Approach
- Uses **Tailwind CSS v4** with CSS variables for theming
- Configured with `tw-animate-css` for animations
- Custom CSS variables defined in globals.css for light/dark themes
- Shadcn/ui "new-york" style with neutral base color

### Key Components & Features
- **Dynamic header**: Changes appearance on scroll (from full-width dark to compact pill-shaped)
- **Project showcase**: Grid layout with hover animations and overlay effects
- **Portfolio theme**: "COLORS AND THE KIDS" creative agency/portfolio site
- **Responsive design**: Mobile-first approach with responsive grid layouts

### Configuration Notes
- TypeScript strict mode enabled
- Next.js config ignores ESLint and TypeScript errors during builds (likely for rapid prototyping)
- Images are unoptimized in Next.js config
- Path mapping: `@/*` maps to root directory
- Shadcn/ui aliases configured for components, utils, ui, lib, and hooks

### Development Patterns
- Uses "use client" directive for client-side interactivity
- Implements scroll-based animations and state changes
- Follows modern React patterns with hooks (useState, useEffect)
- CSS-in-JS styling with Tailwind classes and conditional rendering
- Form handling appears ready for react-hook-form integration (dependency included)

### UI Library Integration
The project includes comprehensive Radix UI components suggesting it's set up for complex UI interactions:
- Dialogs, dropdowns, navigation menus
- Form controls (checkboxes, selects, sliders)
- Data visualization components (recharts)
- Toast notifications (sonner)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/olle556) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
