## nextjs-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
```bash
pnpm dev          # Start development server on http://localhost:3000
pnpm build        # Build for production (creates .next directory)
pnpm start        # Start production server (requires build first)
```

### Code Quality
```bash
pnpm lint         # Run ESLint to check code quality
pnpm format       # Auto-fix ESLint issues and format with Prettier
pnpm check-types  # Run TypeScript type checking without emitting files
```

### Package Management
```bash
pnpm install      # Install dependencies
pnpm add <pkg>    # Add a new dependency
pnpm add -D <pkg> # Add a new dev dependency
pnpm update       # Update dependencies
```

### Git Hooks
Pre-commit hooks are configured via Husky and lint-staged. They automatically run on staged files before commit:
- ESLint fixes for `.js`, `.jsx`, `.ts`, `.tsx` files
- TypeScript type checking
- Prettier formatting for `.json`, `.yaml`, `.md` files

## Architecture

### Provider Hierarchy
The application uses a nested provider structure in the following order:
1. **ReactQueryClientProvider** (`src/app/ReactQueryClientProvider.tsx`) - Wraps entire HTML document, manages server state
2. **Providers** (`src/app/providers.tsx`) - Contains HeroUIProvider and NextThemesProvider for UI components and theme management
3. **Analytics** - Vercel Analytics and Speed Insights are injected at the body level

### Routing Structure
- Uses Next.js 14 App Router (not Pages Router)
- All routes defined in `src/app/` directory
- Layout hierarchy: `layout.tsx` files cascade down the route tree
- API routes in `src/app/api/` using Route Handlers (not API Routes from Pages Router)

### State Management Strategy
- **Server State**: TanStack Query (React Query) with 60-second stale time
- **UI State**: Component-level useState/useReducer
- **Theme State**: next-themes provider
- **Auth State**: Handled via apiClient interceptors (token in localStorage)

### API Client Architecture
The `apiClient` (`src/libs/apis/apiClient.ts`) is a singleton Axios instance with:
- Automatic token injection from localStorage
- 401 response handling with redirect to `/login`
- 10-second timeout
- Base URL from `NEXT_PUBLIC_API_URL` env var

### Styling System
- **Tailwind CSS**: Primary styling method with custom color palette
- **HeroUI**: Component library integrated via Tailwind plugin
- **CSS Variables**: Used for theming (defined in `globals.css`)
- **Dark Mode**: Managed by `next-themes` with class strategy
- Utility function `cn()` for conditional class merging (uses clsx + tailwind-merge)

### TypeScript Configuration
- Strict mode enabled
- Path aliases configured:
  - `@/*` → `src/*`
  - `@/public/*` → `public/*`
- Incremental compilation enabled

### Middleware
Global middleware (`src/middleware.ts`) applies to all routes except API, static files, and images:
- Adds security headers (X-Frame-Options, X-Content-Type-Options, Referrer-Policy)
- Can be extended for authentication, redirects, or request modification

### Component Organization
- `/components/layout/` - Page-level layout components (Header, Footer)
- `/components/ui/` - Reusable UI components (ThemeToggle, etc.)
- `/components/icons/` - Icon components
- Components should be client-side (`'use client'`) when using hooks or browser APIs

### Environment Variables
- Public variables must be prefixed with `NEXT_PUBLIC_`
- Copy `.env.example` to `.env.local` for local development
- Critical variables:
  - `NEXT_PUBLIC_APP_URL`: Application URL
  - `NEXT_PUBLIC_API_URL`: API base URL

## Key Implementation Details

### Metadata Management
Metadata is centralized in `src/app/layout.tsx` using Next.js Metadata API with:
- Dynamic title templates
- OpenGraph and Twitter cards
- Favicon configuration
- Site manifest for PWA support

### Font Loading
Inter font is loaded via `next/font/google` with:
- Latin subset only
- Display swap strategy for performance
- Applied at the HTML element level

### Error Handling
- API errors handled in apiClient interceptors
- 401 errors trigger logout and redirect
- Network timeouts set to 10 seconds

### Build Output
- Static files served from `/public`
- Build output in `.next` directory (gitignored)
- Production builds optimize with CSS nano in PostCSS

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/posgnu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
