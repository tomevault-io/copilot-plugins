## patlehmann1-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal software developer portfolio website built with Next.js 15, TypeScript, and Tailwind CSS. Features a modern design with multi-theme system (including synthwave/retrowave), interactive Three.js particle animations, Apple Music integration, and optimized performance for showcasing projects and technical expertise.

## Tech Stack

- **Next.js 15** with App Router and Turbopack
- **TypeScript** for type safety
- **Tailwind CSS** with custom design system
- **Framer Motion** for animations
- **Three.js + @react-three/fiber** for 3D particle animations
- **@react-three/drei** for Three.js utilities
- **React Hook Form + Zod** for form validation
- **Lucide React** for icons
- **next-themes** for multi-theme system (6 themes: light, dark, ocean, forest, synthwave, minimal)
- **Geist & Geist Mono fonts** for modern typography
- **React Markdown + Remark GFM** for content rendering
- **JSON-based content management** for blog posts
- **RSS & Sitemap generation** for SEO optimization
- **Jest + React Testing Library** for comprehensive testing (814 tests, 93%+ coverage)

## Development Commands

```bash
# Development
npm run dev                 # Start development server with Turbopack
npm run build              # Build for production (includes RSS & sitemap generation)
npm run export             # Export for static hosting (includes RSS & sitemap generation)
npm start                  # Start production server

# Code Quality
npm run lint               # Run linting
npx tsc --noEmit          # Type checking

# Testing
npm test                   # Run all tests
npm run test:watch         # Run tests in watch mode
npm run test:coverage      # Run tests with coverage report
npm run test:ci            # Run tests for CI/CD pipeline

# SEO Generation
npm run generate-rss       # Generate RSS feed
npm run generate-sitemap   # Generate sitemap
npm run generate-seo       # Generate both RSS and sitemap

# Version Management
npm run version:patch      # Bug fixes (1.0.0 → 1.0.1)
npm run version:minor      # New features (1.0.0 → 1.1.0)
npm run version:major      # Breaking changes (1.0.0 → 2.0.0)
```

## Version Management

This project uses **Semantic Versioning (SemVer)** with the format: `MAJOR.MINOR.PATCH`

### When to Update Versions

**IMPORTANT**: Always analyze changes and update the version number before committing significant work.

- **PATCH** (1.0.x) - Bug fixes, typos, small tweaks
  - Fixing broken functionality
  - Correcting typos in content or code
  - Minor style adjustments
  - Performance improvements without API changes

- **MINOR** (1.x.0) - New features, backwards compatible
  - Adding new components or pages
  - New blog posts or content
  - Enhanced functionality that doesn't break existing features
  - New configuration options

- **MAJOR** (x.0.0) - Breaking changes
  - Removing or significantly changing public APIs
  - Major architectural changes
  - Dependency updates that change behavior
  - Changes requiring user intervention

### Version Bump Workflow

```bash
# Analyze your changes first, then run the appropriate command:
npm run version:patch      # Creates commit, tag, and pushes automatically
npm run version:minor      # Creates commit, tag, and pushes automatically
npm run version:major      # Creates commit, tag, and pushes automatically
```

These commands automatically:
1. Update package.json version
2. Create a git commit with the version bump
3. Create an annotated git tag
4. Push changes and tags to remote

### Manual Version Management

If you need more control:
```bash
npm version patch -m "Fix: description"     # Update version and commit
git push && git push --tags                 # Push changes and tags
```

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── blog/              # Blog pages and layout
│   │   ├── [slug]/        # Dynamic blog post pages
│   │   ├── layout.tsx     # Blog layout component
│   │   ├── page.tsx       # Blog listing page
│   │   └── blog-client.tsx # Client-side blog components
│   ├── globals.css        # Global styles with Tailwind CSS
│   ├── layout.tsx         # Root layout component
│   └── page.tsx           # Home page
├── components/
│   ├── ui/                # Reusable UI components (Button, ThemeToggle)
│   ├── sections/          # Page sections (Hero, About, Projects, Blog)
│   ├── layout/            # Layout components (Navigation, sub-components)
│   ├── blog/              # Blog-specific components (BlogCard, BlogContent, ReadingProgressBar)
│   └── newsletter/        # Newsletter signup components
├── hooks/                 # Custom React hooks
│   ├── useScrollToSection.ts # Navigation with smooth scrolling
│   ├── useBlogFiltering.ts   # Blog filtering logic
│   └── useReadingProgress.ts # Reading progress tracking
├── lib/
│   ├── types.ts           # TypeScript type definitions and utility types
│   ├── utils.ts           # Utility functions (cn helper, formatters)
│   ├── ui-utils.ts        # UI-specific utility functions
│   ├── constants.ts       # Application constants and configuration
│   ├── blog.ts            # Blog data handling and utilities
│   └── rss.ts             # RSS feed generation
├── content/
│   ├── projects/          # Project data and content
│   └── blog/
│       └── articles.json  # Blog posts data (JSON format)
scripts/                   # Build and generation scripts
├── generate-rss.js        # RSS feed generation script
└── generate-sitemap.js    # Sitemap generation script
docs/                       # Detailed documentation
├── architecture.md        # System design and architecture details
├── testing.md             # Comprehensive testing guide
├── development-guidelines.md # Code quality and best practices
└── code-review-checklist.md # Review requirements and checklists
public/                     # Static assets (includes generated RSS & sitemap)
jest.config.js             # Jest testing configuration
setupTests.ts              # Global test setup and mocks
```

## Key Features

### ✅ Implemented Features
1. **Hero Section** - Animated introduction with social links and typewriter effect
2. **Multi-Theme System** - 6 themes (light, dark, ocean, forest, synthwave, minimal) with system preference detection
3. **Synthwave/Retrowave Theme** - Electric neon color palette with custom gradients and glow effects
4. **Three.js Particle Background** - Interactive particle system with 60 animated particles and mouse tracking
5. **Apple Music Integration** - Embedded "Lehmann Dev Mix" playlist in About section
6. **Typewriter Effect** - Rotating developer titles with Geist Mono monospace font
7. **Projects Showcase** - Interactive grid with filtering
8. **Blog Section** - JSON-powered articles with full blog layout
9. **Contact Form** - React Hook Form with Zod validation
10. **Newsletter Subscription** - Kit.com integration with form validation and custom domain (newsletter.patricklehmann.io)
11. **SEO Optimization** - Metadata API, structured data, RSS feeds, and sitemap
12. **RSS Feed Generation** - Automated feed generation for blog posts
13. **Sitemap Generation** - SEO-optimized sitemap with proper metadata
14. **Blog Navigation** - Enhanced navigation with active link styling
15. **Content Management** - JSON-based blog system with filtering and search
16. **Reading Progress Indicator** - Smooth animated progress bar for blog posts
17. **Hydration-Safe Theme Rendering** - Mounted state pattern prevents SSR/client mismatches
18. **Comprehensive Jest Testing Suite** - 708 tests with 93%+ coverage

### 🚀 Potential Future Enhancements
- Blog post comments system
- Advanced blog filtering (by date range, multiple tags)
- Blog post series/categories
- Related posts suggestions

## 📖 Detailed Documentation

For comprehensive information about specific aspects of the project, refer to these detailed guides:

- **[🏗️ Architecture & System Design](docs/architecture.md)** - Component system, styling approach, blog architecture, SEO implementation, and technical details
- **[🧪 Testing Guide](docs/testing.md)** - Jest configuration, testing patterns, coverage requirements, and testing best practices
- **[📝 Development Guidelines](docs/development-guidelines.md)** - Code quality standards, React/Next.js/TypeScript best practices, and anti-patterns to avoid
- **[✅ Code Review Checklist](docs/code-review-checklist.md)** - Comprehensive review requirements, quality checks, and approval criteria

## Quick Reference

### Essential Guidelines
- **Readability**: Write self-documenting code with clear naming and structure
- **Maintainability**: Keep components focused, use composition, follow DRY principles
- **YAGNI Compliance**: Implement only what's needed, avoid over-engineering
- **Type Safety**: Use strict TypeScript, avoid `any`, implement proper interfaces
- **Testing**: Write tests for new features, maintain 70% coverage minimum

### Quality Gate Checks
**IMPORTANT**: Always run quality-gate checks after making any code changes. These checks are mandatory before considering any work complete.

Run all three checks in parallel for efficiency:
```bash
npm test && npx tsc --noEmit && npm run lint
```

Or run individually:
- `npm test` - Verify all tests pass (708 tests expected)
- `npx tsc --noEmit` - Ensure no TypeScript errors
- `npm run lint` - Confirm no linting issues

### Before Committing
- [ ] Run `npm test` - All tests pass
- [ ] Run `npx tsc --noEmit` - No TypeScript errors
- [ ] Run `npm run lint` - No linting issues
- [ ] Analyze changes and update version if needed (`npm run version:patch/minor/major`)
- [ ] Review relevant checklist items in [Code Review Checklist](docs/code-review-checklist.md)

### Architecture Patterns
- **Component Organization**: Domain-based structure (ui/, sections/, layout/, blog/)
- **State Management**: Local state with custom hooks, context for global state
- **Styling**: Tailwind CSS with design system tokens, `cn()` utility for conditional classes
- **Testing**: React Testing Library for components, Jest for utilities, comprehensive mocking

### Content & Writing Guidelines
- **Natural Language**: Write conversationally, avoid AI-generated patterns and corporate buzzwords
- **Avoid Repetitive Structures**: No "Whether it's X, Y, or Z" patterns or forced analogies
- **Simple Punctuation**: Prefer periods and commas over em dashes and complex punctuation
- **Personal Voice**: Sound human and authentic, not like marketing copy or AI-generated content
- **Direct Communication**: Use short, clear sentences instead of complex, verbose structures

### Comment Guidelines
- **Avoid unnecessary comments**: Write self-documenting code with clear naming and structure
- **Only comment complex functions**: Add JSDoc comments above functions that require explanation of their purpose, parameters, or complex logic
- **Remove obvious comments**: Don't comment what the code clearly shows (e.g., `// Set loading state`)
- **No section headers**: Avoid organizational comments like `// Constants` or `// Helper functions`
- **Preserve valuable documentation**: Keep comments that explain non-obvious business logic, complex algorithms, or important context

For detailed implementation patterns, architectural decisions, and comprehensive guidelines, please refer to the linked documentation files above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/patlehmann1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
