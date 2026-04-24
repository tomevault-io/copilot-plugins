## deakinrover-space

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm start` - Start production server
- `npm run lint` - Run ESLint
- Installation requires `npm install --legacy-peer-deps` or `yarn install --legacy-peer-deps`

## Project Architecture

This is a modern Next.js 14 space-themed portfolio website with 3D animations and interactive elements.

### Key Technologies
- **Next.js 14** with App Router
- **TypeScript** for type safety
- **Tailwind CSS** for styling
- **Three.js** with React Three Fiber for 3D graphics
- **Framer Motion** for animations
- **React Intersection Observer** for scroll-based effects

### Project Structure
- `app/` - Next.js 14 App Router pages and layouts
- `components/main/` - Main page sections (navbar, hero, skills, projects, footer)
- `components/sub/` - Reusable sub-components
- `constants/index.ts` - Static data (skills, projects, social links, navigation)
- `config/index.ts` - Site metadata configuration
- `lib/` - Utility functions and motion configurations
- `public/` - Static assets organized by type (projects/, skills/, videos/)

### Component Architecture
- Layout uses fixed navbar with responsive hamburger menu
- StarsCanvas provides animated starfield background
- Main sections: Hero, Skills (categorized), Projects, Footer
- All data is centralized in `constants/index.ts` as const assertions

### Styling Patterns
- Uses `cn()` utility (clsx + tailwind-merge) for conditional classes
- Space theme with dark background (`#030014`)
- Purple accent color (`rgb(112,66,248)`)
- Responsive design with mobile-first approach
- Custom backdrop-blur effects for glassmorphism

### TypeScript Configuration
- Strict mode enabled
- Path aliases: `@/*` maps to root directory
- Includes Next.js plugin for enhanced TypeScript support

### Content Management
- Skills data categorized by Frontend/Backend/Fullstack/Other
- Projects with title, description, image, and link
- Social links and footer data all configurable in constants
- Navigation links for smooth scrolling to sections

### Development Notes
- Install dependencies with `--legacy-peer-deps` flag
- Logo currently uses `/deakin_rover_logo_white.svg`
- All images should be optimized for web (skills icons are 80x80px typically)
- Three.js animations require client-side rendering (`'use client'`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mirmisbahali) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
