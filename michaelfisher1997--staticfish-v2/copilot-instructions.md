## staticfish-v2

> Guidelines for AI agents working on this Astro-based static site.

# AGENTS.md

Guidelines for AI agents working on this Astro-based static site.

## Build Commands

```bash
# Development (Cloudflare dev server, port 5050)
bun dev

# Production build (uses astro.prod.config.mjs)
bun build

# Preview production build
bun preview

# Start production server
bun start
```

**Critical:** Production builds must use `astro.prod.config.mjs` (not `astro.config.mjs`) which contains the `react-dom/server.edge` alias for Cloudflare Workers.

## Testing

**No test framework is currently configured.** This project does not have unit tests, integration tests, or E2E tests. If adding tests:

- Consider Vitest for unit tests (compatible with Astro/Vite)
- Consider Playwright for E2E tests
- Run single test: `bun test <pattern>` (once configured)

## Linting

The project uses **oxlint** (not ESLint). Check for linting issues:

```bash
# Check all files
npx oxlint

# Check specific file
npx oxlint src/components/Button.tsx
```

## Code Style Guidelines

### File Organization
```
src/
├── components/ui/     # Reusable UI components (button, card, input, badge)
├── components/        # Page-specific components (Header, Footer, MobileMenu)
├── layouts/           # Astro layouts
├── pages/             # Astro pages (file-based routing)
├── lib/               # Utilities and helpers
├── styles/            # Global CSS
├── data/              # Static data files (JSON)
└── env.d.ts           # TypeScript declarations
```

### Naming Conventions
- **Components:** PascalCase (e.g., `Button.tsx`, `Header.astro`)
- **Utilities:** camelCase (e.g., `utils.ts`)
- **Pages:** lowercase with dashes (e.g., `privacy.astro`)
- **CSS Classes:** Use Tailwind, avoid custom CSS when possible
- **Variables:** camelCase for TS/JS, kebab-case for CSS custom properties

### Imports (Order Matters)
1. React/ Astro imports
2. Third-party libraries
3. Local components
4. Local utilities
5. Types

Example:
```typescript
import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "../../lib/utils";
```

### Component Patterns

**UI Components (CVA Pattern):**
```typescript
import { cva, type VariantProps } from "class-variance-authority";

const buttonVariants = cva(
  "base-classes",
  {
    variants: {
      variant: { default: "...", outline: "..." },
      size: { default: "...", sm: "...", lg: "..." },
    },
    defaultVariants: { variant: "default", size: "default" },
  }
);

export interface ButtonProps extends VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}
```

**Styling:**
- Always use `cn()` utility for class merging: `cn(buttonVariants({ variant, size, className }))`
- Use Tailwind's HSL color system (e.g., `text-primary`, `bg-slate-900`)
- Avoid arbitrary values; extend `tailwind.config.js` if needed

### TypeScript
- **Strict mode enabled** - no implicit any
- Use explicit return types on exported functions
- Prefer `interface` over `type` for object shapes
- Use `type` for unions and utility types
- React components: `React.FC<Props>` or explicit return type

### Error Handling
- Use early returns for guard clauses
- Don't swallow errors; log them or propagate
- For async operations, use try/catch with proper error messages

### Astro-Specific
- Use `client:*` directives sparingly (React components only when needed)
- Prefer static generation over SSR where possible
- Use `Astro.props` with proper typing in layouts

### Environment Variables
- Sanity supports both `PUBLIC_SANITY_*` and `SANITY_*` prefixes
- Client uses `useCdn: false` for fresh data
- **Critical:** Different project IDs in `sanity.config.js` vs `astro.config.mjs`

### Security
- CSP headers auto-injected via `src/middleware.ts`
- Runtime nonce generation for inline scripts
- Strict headers: CSP, HSTS, X-Frame-Options

### Docker
- Base image: `oven/bun:latest` (NOT node:alpine)
- Exposed port: 5050
- Dev server: `bun run dev -- --host --port 5050`

### Git Workflow
- Commit messages: concise, describe "why" not "what"
- Use GitHub Actions for CI (see `.github/workflows/opencode.yml`)
- Trigger opencode with `/oc` or `/opencode` in PR/issue comments

## Common Patterns

**Button Usage:**
```astro
<Button size="lg" className="rounded-full">
  <a href="/contact">Get Started</a>
</Button>
```

**Glass Card:**
```html
<div class="glass-card p-8 rounded-4xl">...</div>
```

**Responsive Grid:**
```html
<div class="grid grid-cols-1 md:grid-cols-3 gap-8">...</div>
```

## Dependencies to Know
- `class-variance-authority` - Component variants
- `tailwind-merge` + `clsx` - Class merging via `cn()`
- `lucide-react` - Icons
- `astro` + `@astrojs/react` + `@astrojs/tailwind`
- `@astrojs/cloudflare` - Cloudflare adapter

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelFisher1997)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MichaelFisher1997)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
