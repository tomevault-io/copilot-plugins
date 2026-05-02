## halfguru-github-io

> This document provides guidance for agentic coding agents working in this Astro + Tailwind CSS portfolio repository.

# AGENTS.md - Development Guide for Agentic Coding

This document provides guidance for agentic coding agents working in this Astro + Tailwind CSS portfolio repository.

## Build & Development Commands

```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run preview      # Preview production build locally
npm ci               # Install dependencies
npx astro check      # Type checking
npm run lint         # Linting
npm run lint:fix     # Auto-fix linting issues
```

**Note**: No test framework is configured. Do not run test commands.

## Tech Stack

- **Astro**: Static site generator with island architecture
- **Tailwind CSS**: Utility-first CSS via @tailwindcss/vite
- **TypeScript**: Strict mode (extends astro/tsconfigs/strict)
- **Vite**: Build tool and dev server

## Code Style Guidelines

### File Structure

- `.astro` files for components and pages
- TypeScript logic in Astro frontmatter (between `---` fences)
- Components: `src/components/` | Pages: `src/pages/` | Styles: `src/styles/`
- Site configuration: `src/config.ts`

### TypeScript

- Strict mode enabled - type all new code properly
- Type inference preferred when types are clear
- Use `@ts-expect-error` with comment for known type issues (see astro.config.mjs)

### Linting

- ESLint with flat config (eslint.config.mjs)
- Uses TypeScript strict rules: `...tseslint.configs.strict`
- Rules: `@typescript-eslint/no-explicit-any: warn`, unused vars with `_` prefix ignored
- Run `npm run lint` before committing

### Styling with Tailwind

- Use Tailwind utility classes exclusively
- Responsive prefixes: `sm:`, `md:`, `lg:`, `xl:` (mobile-first)
- Dynamic accent color via inline styles: `style={`background-color: ${siteConfig.accentColor}`}`
- CSS variables for hover states: `hover:text-[var(--accent-color)]`
- Semantic spacing: `p-8`, `gap-4`, `mt-6`

### Adding New Features

1. Update `src/config.ts` with new data
2. Import config: `import { siteConfig } from "../config"`
3. Use Tailwind classes for styling
4. Test responsiveness with breakpoints
5. Run: `npx astro check && npm run lint && npm run build`

## Deployment

- GitHub Actions: `.github/workflows/deploy.yml`
- Runs lint and type check before building
- Deploys to GitHub Pages on push to `main`
- Node.js 18+ required

## Config Structure

The `src/config.ts` exports a single `siteConfig` object containing:
- `name`, `title`, `description` - Site metadata
- `accentColor` - Primary color (hex)
- `social` - Social links (email, linkedin, twitter, github)
- `aboutMe` - Bio text
- `skills` - Array of skill strings
- `projects` - Array of { name, description, link?, skills[] }
- `experience` - Array of { company, title, dateRange, bullets[] }
- `education` - Array of { school, degree, dateRange, achievements[] }

---
> Source: [halfguru/halfguru.github.io](https://github.com/halfguru/halfguru.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
