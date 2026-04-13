## portfolio-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Commands
```bash
pnpm dev            # Start development server
pnpm build          # Build production (runs lint first)
pnpm start          # Start production server
pnpm lint           # Run ESLint
```

### Testing
```bash
pnpm test           # Run all tests once
pnpm test:watch     # Run tests in watch mode
pnpm test:coverage  # Run tests with coverage report
```

## Architecture Overview

This is a Next.js 15 portfolio website built with TypeScript, React 19, and Tailwind CSS. The architecture follows modular design principles with clear separation of concerns.

### Key Technical Stack
- **Framework**: Next.js 15 with App Router
- **Language**: TypeScript with strict configuration
- **Styling**: Tailwind CSS v4 with PostCSS integration
- **UI Components**: Radix UI primitives with custom variants
- **Testing**: Vitest with jsdom environment
- **Code Quality**: ESLint with TypeScript, stylistic, and import rules
- **Fonts**: Geist Sans and Geist Mono
- **Deployment**: Vercel with automatic optimization

### Project Structure

```
app/                    # Next.js app router pages
├── blog/              # Blog functionality
├── globals.css        # Global styles
├── layout.tsx         # Root layout with metadata
└── page.tsx           # Homepage

components/            # React components organized by domain
├── blog/             # Blog-specific components
├── cards/            # Card components (Project, Service)
├── layout/           # Layout components (Header, Footer, Navigation)  
├── sections/         # Page sections (Hero, About, etc.)
└── ui/               # Reusable UI primitives

config/               # Configuration files with TypeScript interfaces
├── blog.ts           # Blog configuration and types
├── experience.ts     # Experience/career data
├── portfolio.ts      # Portfolio projects data
└── services.ts       # Services offered data

content/              # Markdown content
└── blog/             # Blog posts in markdown format

lib/                  # Utilities and shared logic
├── markdownProcessor.ts  # Custom markdown processing
└── utils.ts          # Utility functions (includes cn helper)
```

### Component Architecture

Components follow a consistent pattern:
- **Domain-organized**: Components grouped by feature/domain (blog, cards, sections)
- **Co-located tests**: Each component has a `.test.tsx` file alongside it
- **TypeScript interfaces**: Props interfaces named simply as `Props`
- **Arrow function components**: Using `FC<Props>` pattern
- **Radix UI integration**: UI primitives use class-variance-authority for variants

### Content Management

Blog content is managed through:
- **Markdown files** in `content/blog/` directory
- **Custom processor** (`lib/markdownProcessor.ts`) for parsing
- **Syntax highlighting** via react-syntax-highlighter
- **Type-safe configuration** in `config/blog.ts`

### Code Style Standards

The project enforces strict coding standards:
- **Single quotes** for strings
- **4-space indentation**
- **Semicolons required**
- **No default exports** (except Next.js pages which require them)
- **One React component per file** with matching filename
- **Strict TypeScript** with type checking
- **Import organization** with typescript resolver

### Testing Strategy

- **Vitest** as test runner with jsdom environment
- **Testing Library** for React component testing  
- **Co-located tests** alongside source files
- **Behavior-focused testing**: Tests focus on component behavior vs implementation details
- **Coverage reporting** with v8 provider
- **Test-specific ESLint rules** that relax strict typing for testing

### Build Configuration

- **Strict ESLint** runs before build
- **TypeScript checking** enforced during build
- **Next.js optimizations**: Compression, React strict mode, no powered-by header
- **Vercel deployment** with automatic image optimization and static generation

### Development Philosophy

Based on the comprehensive DEVELOPMENT_GUIDE.md, this project follows:
- **YAGNI principle**: Only implement what's needed
- **Component-based architecture**: Clear separation of concerns
- **Test-driven development**: Tests co-located with components
- **Modular design**: Self-contained, focused modules
- **Performance-conscious**: Optimized builds and careful dependency management

## Development Guidelines

This project follows the comprehensive development standards outlined in DEVELOPMENT_GUIDE.md. Key guidelines include:

### Component Standards
- **One React component per file** with matching filename (e.g., `UserCard.tsx` contains `UserCard` component)
- **Arrow function components**: Use `FC<Props>` pattern with arrow functions
- **Simple Props interface**: Name component props interface as `Props`
- **Co-located tests**: Each component should have a `.test.tsx` file alongside it
- **Domain organization**: Group components by feature/domain rather than by technical type

### File Organization
- **Keep files under 300 lines** for maintainability
- **Tests co-located** with source code using `.test.tsx` extension
- **Private methods at the end** of each file
- **Export at definition**: Use `export const ComponentName` instead of separate export statements

### TypeScript Standards
- **Let TypeScript infer types** when obvious (e.g., return types)
- **Explicitly type** complex objects and function parameters
- **No default exports** except for Next.js pages that require them
- **Avoid `any`, `as`, `!`** assertions in production code
- **Use interfaces for data models**, classes for behavior

For complete development standards, testing strategies, and architectural patterns, refer to DEVELOPMENT_GUIDE.md.

## Important Notes

- Build process includes linting, so code must pass ESLint to build successfully
- All components should have accompanying tests focused on behavior
- Navigation logic is separated into dedicated Navigation component
- Markdown content processing uses custom implementation, not external libraries
- Font loading is handled through Next.js with Geist font family
- The project uses pnpm as package manager, not npm or yarn
- Components use consolidated structure to reduce over-decomposition

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ch1ll0ut1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ch1ll0ut1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
