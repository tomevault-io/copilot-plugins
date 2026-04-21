## portfolio

> This file provides guidance to cursor when working with code in this repository.

This file provides guidance to cursor when working with code in this repository.

## Development Commands

This is a Next.js portfolio project. Use these commands for development:

- `pnpm dev` - Start development server
- `pnpm dev:turbo` - Start development server with Turbopack
- `pnpm build` - Build for production
- `pnpm start` - Start production server
- `pnpm lint` - Run ESLint for code quality

## Project Architecture

### Core Technologies
- **Next.js 15** with App Router
- **TypeScript** with strict mode
- **Tailwind CSS** for styling
- **MDX** for content management (blog posts and case studies)
- **GSAP** and **Motion** (previously Framer Motion) for animations and interactions
- **Three.js** and React Three Fiber for 3D graphics
- **Lenis** for smooth scrolling

### Key Directory Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── (experiments)/      # Experimental features/demos
│   ├── blog/              # Blog/journal functionality
│   └── about/             # About page
├── components/            # Reusable UI components
│   ├── animations/        # GSAP animation components
│   ├── blog-collection/   # Blog-specific components
│   ├── image-reveal-threejs/ # Three.js shader components
│   └── ui/               # Base UI components (shadcn/ui)
├── modules/              # Feature-specific modules
├── lib/                  # Utilities and helpers
├── hooks/                # Custom React hooks
└── types/                # TypeScript type definitions

content/                  # MDX blog posts and case studies
public/images/           # Static images organized by feature
```

### Architecture Patterns

**Component Organization**: Components are organized by feature in `modules/` for complex functionality, and `components/` for reusable UI elements.

**Content Management**: Blog posts and case studies are written in MDX format in the `content/` directory. The blog system uses `gray-matter` for frontmatter parsing and supports dynamic routing via `[slug]` pages.

**Animation System**: The project uses both GSAP for complex animations with React integration via `@gsap/react`, and Motion (previously Framer Motion) for React-specific animations and interactions. Three.js components use `@react-three/fiber` and `@react-three/drei` for 3D graphics.

**Styling**: Tailwind CSS with custom font variables defined in `layout.tsx`. Uses CSS-in-JS for component-specific styling where needed.

**Experiments**: The `(experiments)` route group contains interactive demos and prototypes for various UI patterns and animations.

## Key Configuration

### Next.js Configuration
- MDX support enabled with `@next/mdx`
- Turbopack rules for GLSL shaders and SVG processing
- Package optimization for Three.js and GSAP libraries

### Path Aliases
- `@/*` maps to `src/*` for clean imports

### Content Processing
- MDX files processed with custom components
- GLSL shaders supported via glslify-loader
- SVG files processed with @svgr/webpack

## Special Features

**Shader Integration**: Custom GLSL shaders in `components/image-reveal-threejs/` with includes for reusable shader functions.

**Interactive Experiments**: Each experiment in the `(experiments)` directory showcases different interaction patterns and animation techniques.

**Blog System**: Dynamic blog with slug-based routing, MDX content, and custom blog components for rich content presentation.

**Performance**: Uses experimental package optimization and proper image optimization throughout the project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juliorafre) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
