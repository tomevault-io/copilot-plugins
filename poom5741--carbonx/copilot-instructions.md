## carbonx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Blockedge Exchange is a cryptocurrency trading platform frontend built with React 19, TypeScript, Vite 7, and Tailwind CSS. The application uses shadcn/ui components built on Radix UI primitives.

**Key Technologies:**
- React 19 + TypeScript ~5.9.3
- Vite 7 (build tool)
- React Router DOM 7 (routing)
- Tailwind CSS 3 (styling)
- Radix UI primitives + shadcn/ui (component library)
- Recharts + Lightweight Charts (data visualization)
- GSAP (animations)
- Vitest + Testing Library (testing)

## Development Commands

All commands must be run from the `frontend/` directory:

```bash
# Install dependencies
pnpm install

# Start development server (http://localhost:5173)
pnpm dev

# Type check
pnpm tsc -b

# Build for production
pnpm build

# Run linter
pnpm lint

# Run tests
pnpm test
pnpm test:ui          # Run with Vitest UI
pnpm test:coverage    # Run with coverage report
```

**Node Version:** Requires Node.js 20.19+ or 22.12+ (uses `.nvmrc` with Node 22.14.0 LTS)

## Architecture

### Directory Structure
```
frontend/
├── src/
│   ├── components/
│   │   ├── ui/           # shadcn/ui components (Radix UI primitives)
│   │   ├── portfolio/    # Portfolio feature components
│   │   ├── Navbar.tsx
│   │   └── LoginModal.tsx
│   ├── pages/            # Route components (Landing, Trading, Markets, Portfolio)
│   ├── hooks/            # Custom React hooks
│   ├── lib/              # Utility functions
│   ├── utils/            # Utilities (includes cn() class merge utility)
│   ├── App.tsx           # Main app with routing
│   ├── main.tsx          # Entry point
│   └── vitest.setup.ts   # Test setup
├── public/               # Static assets
├── vite.config.ts        # Vite configuration
├── vitest.config.ts      # Vitest configuration
└── vercel.json          # Vercel deployment config
```

### Routing Structure

The app uses React Router DOM with the following routes (defined in [App.tsx](frontend/src/App.tsx)):
- `/` - LandingPage
- `/trade` - TradingPage
- `/markets` - MarketsPage
- `/portfolio` - PortfolioPage
- `/*` - NotFound404 (catch-all 404 page)

**Note:** Routes are standardized to lowercase without trailing slashes (e.g., `/markets` not `/Markets`).

### Authentication State

Authentication is managed at the top level in [App.tsx](frontend/src/App.tsx:12) with state passed down to page components:
- `isLoggedIn` - Boolean auth state
- `showLoginModal` - Controls LoginModal visibility
- `handleLogin` / `handleLogout` - Auth handlers

All page components receive `isLoggedIn` and `onLoginClick` props to trigger the login modal.

### Component Organization

**UI Components** ([components/ui/](frontend/src/components/ui/)): shadcn/ui components built on Radix UI. These are headless primitives with full accessibility support.

**Feature Components**: Organized by feature (e.g., [components/portfolio/](frontend/src/components/portfolio/)) with co-located tests using `.test.tsx` suffix.

### Import Aliases

Path aliases are configured via `@/` prefix:
- Vite: [vite.config.ts](frontend/src/vite.config.ts:12)
- TypeScript: [tsconfig.app.json](frontend/src/tsconfig.app.json.json:13)

Example import: `import { cn } from '@/lib/utils'`

### Testing Setup

Vitest is configured with:
- Environment: jsdom
- Setup file: [src/vitest.setup.ts](frontend/src/vitest.setup.ts)
- Coverage provider: v8
- Pool options: forked with single thread (singleThread: true, isolate: false)
- Global test API: `vitest/globals` enabled in [tsconfig.app.json](frontend/src/tsconfig.app.json.json:8)

Test files: `*.test.tsx`, `*.test.ts`, `*.spec.tsx`, `*.spec.ts`

### Styling Conventions

**Class Merging:** Use the `cn()` utility from [lib/utils.ts](frontend/src/lib/utils.ts:4) for conditional Tailwind classes:
```tsx
import { cn } from '@/lib/utils'
className={cn('base-class', isActive && 'active-class')}
```

**Component Styling:** All UI components use Tailwind CSS. Dark mode is default (base background: `bg-[#0a0e17]`).

### Linting

ESLint is configured with:
- TypeScript ESLint recommended rules
- React Hooks ESLint rules
- React Refresh for Vite HMR
- Config: [eslint.config.js](frontend/src/eslint.config.js)
- Ignores: `dist/`

### Build Output

- Build command runs TypeScript compiler check (`tsc -b`) then Vite build
- Output directory: `dist/`
- For Vercel deployment, [vercel.json](frontend/src/vercel.json) configures build settings and SPA routing

### Deployment

The app is configured for Vercel deployment with SPA routing support. See [vercel.json](frontend/src/vercel.json:5) for rewrites configuration that handles client-side routing.


## Planning Rules
- **Summary Only:** Plans must be high-level checklists, not detailed technical specs.
- **No Code in Plans:** Never include code blocks or implementation details in a plan.md file.
- **Limit Depth:** Break plans into phases. Do not plan more than 3-5 steps ahead.
- **File Size:** If a plan exceeds 100 lines, stop and ask the user to break the task down.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Poom5741)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Poom5741)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
