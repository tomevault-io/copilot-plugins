## xma-website

> Instructions for AI coding agents working in this repository.

# AGENTS.md

Instructions for AI coding agents working in this repository.

## Build/Lint/Test Commands

```bash
bun dev          # Start dev server at localhost:3000
bun build        # Production build
bun lint         # Run ESLint
bun knip         # Detect unused exports and dependencies
```

**Package Manager**: Always use `bun`, never `npm` or `yarn`.

**No test suite configured** - this project does not have automated tests.

## Tech Stack

- **Framework**: Next.js 15 with App Router
- **UI**: React 19 RC, Radix UI primitives, class-variance-authority (CVA)
- **Styling**: Tailwind CSS 3.4 with oklch color system
- **Animation**: Framer Motion via `motion/react`
- **CMS**: Contentful (blog content)
- **Analytics**: PostHog (EU), Vercel Analytics, Google Tag Manager
- **Payments**: Stripe
- **Icons**: Phosphor React (prefer `weight="duotone"`), Lucide React

## Project Structure

```
app/
├── (marketing)/          # Marketing pages with shared layout
│   ├── page.tsx          # Home page (server component)
│   └── _components/      # Co-located route components
├── (booking)/            # Booking flow pages
├── (legal)/              # Legal pages
├── blog/                 # Contentful-powered blog
├── services/             # Service pages
└── api/                  # API routes

components/
├── layout/               # Header, Footer, navigation
├── ui/                   # Design system primitives
└── providers/            # Context providers

lib/                      # Shared utilities (cn, posthog-events)
hooks/                    # Custom React hooks
utils/                    # Server utilities (stripe, cloudinary)
data/                     # Static data files
```

## Code Style Guidelines

### File Organization (Locality of Behavior)

Co-locate feature-specific code with routes using underscore-prefixed folders:
- `_components/` - Route-specific components
- `_hooks/` - Route-specific hooks
- `_lib/` - Route-specific utilities
- `_types/` - Route-specific types

Only truly shared code belongs in root-level directories.

### Import Order

1. External packages (`react`, `next/*`, third-party)
2. Absolute imports (`@/components/*`, `@/lib/*`, `@/hooks/*`)
3. Relative imports (`./`, `../`)

```tsx
import React from "react";
import { motion } from "motion/react";  // NOT "framer-motion"
import Link from "next/link";

import { cn } from "@/lib/utils";
import { ScanningButton } from "@/components/ui/ScanningButton";

import { LocalComponent } from "./_components/LocalComponent";
```

### TypeScript

- Strict mode enabled
- Use `interface` for component props, `type` for unions/intersections
- Prefer explicit return types for exported functions
- Use `Record<string, unknown>` over `any` where possible

### Component Patterns

**Server/Client Split**: Server component for metadata, client for interactivity:
```tsx
// app/example/page.tsx (server)
export const metadata: Metadata = { ... };
export default function Page() {
  return <PageClient />;
}

// app/example/_components/PageClient.tsx (client)
"use client";
export default function PageClient() { ... }
```

**CVA for Variants**: All UI primitives use class-variance-authority:
```tsx
const buttonVariants = cva("base-classes", {
  variants: { size: { sm: "...", md: "..." } },
  defaultVariants: { size: "md" },
});
```

**forwardRef Pattern**: For reusable components:
```tsx
const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, ...props }, ref) => (
    <button ref={ref} className={cn(baseClass, className)} {...props} />
  )
);
Button.displayName = "Button";
```

### Naming Conventions

- **Components**: PascalCase (`HeroSection.tsx`)
- **Hooks**: camelCase with `use` prefix (`useTrackEvent.ts`)
- **Utilities**: camelCase (`utils.ts`)
- **Constants**: SCREAMING_SNAKE_CASE (`POSTHOG_EVENTS`)
- **CSS classes**: kebab-case (`glass-primary`)

### Error Handling

API routes pattern:
```tsx
export async function POST(req: Request) {
  try {
    // logic
    return NextResponse.json({ data });
  } catch (error: unknown) {
    console.error("Context:", error);
    return NextResponse.json({ error: "message" }, { status: 500 });
  }
}
```

## Design System Constraints

### Colors (CRITICAL)

**Never use hex or rgb. Always use oklch or HSL via CSS variables.**

```tsx
// WRONG
className="bg-[#3b82f6]"
className="bg-[rgb(59,130,246)]"

// CORRECT
className="bg-blue-500"
className="text-blue-400"
className="bg-primary"
```

Primary palette: `blue-50` through `blue-950` (oklch-based)
Semantic: `primary`, `neutral`, `success`, `warning`, `error`

### Typography

- Display headings: `font-display` (Cormorant Garamond serif)
- Body text: `font-sans` (Inter)

### Glass Effects

Cards (NO backdrop blur):
- `glass-primary`, `glass-secondary`, `glass-tertiary`

Navigation/floating (WITH backdrop blur):
- `glass-nav`, `glass-float`

### Layout

- Container: `max-w-7xl mx-auto`
- Section spacing: `section-padding` or `py-12 lg:py-24 px-4`
- **Never use `overflow-hidden` with sticky positioning**

### Buttons

Primary CTA: `<ScanningButton>` from `components/ui/ScanningButton.tsx`
Generic: `<Button>` from `components/ui/button.tsx`

## Key Patterns

### Event Tracking

Use hooks from `hooks/useTrackEvent.ts`:
```tsx
const trackCTA = useTrackCTA();
trackCTA("Button Text", "section_name", { extra: "props" });
```

### Motion Animations

```tsx
import { motion } from "motion/react";  // CORRECT import path

<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.6 }}
>
```

### Icons

Prefer Phosphor with duotone weight:
```tsx
import { CaretDown } from "phosphor-react";
<CaretDown weight="duotone" className="w-4 h-4" />
```

## Environment Variables

Required:
- `CONTENTFUL_SPACE_ID`, `CONTENTFUL_ACCESS_TOKEN`
- `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`
- `NEXT_PUBLIC_POSTHOG_KEY`, `NEXT_PUBLIC_POSTHOG_HOST`

## Provider Hierarchy

```
ThemeProvider → Theme (Radix) → QueryProvider → PostHogProvider → GlobalAnalyticsProvider
```

## Restrictions

1. **No ambient glow decorations**
2. **No emojis** unless explicitly requested
3. **Minimize comments** - use expressive function/variable names
4. **No hardcoded values** - use design tokens
5. **Dark mode first** - design for dark, adapt for light
6. **Componentize** - pages should not be long monolithic files

---
> Source: [XMA-Faez/xma-website](https://github.com/XMA-Faez/xma-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
