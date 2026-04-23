## ghosthands

> ├── app/                    # Next.js App Router

# Project Structure

## Directory Organization

```
├── app/                    # Next.js App Router
│   ├── fonts/             # Custom font files and configuration
│   ├── about/             # About page route
│   ├── works/             # Works showcase routes
│   │   └── [id]/          # Dynamic work detail pages
│   ├── globals.css        # Global styles and CSS variables
│   ├── layout.tsx         # Root layout component
│   └── page.tsx           # Home page component
├── components/            # Reusable React components
│   ├── ui/               # shadcn/ui components (auto-generated)
│   └── *.tsx             # Custom application components
├── hooks/                # Custom React hooks
├── lib/                  # Utility functions and configurations
├── public/               # Static assets
│   └── images/           # Image assets organized by category
└── styles/               # Additional stylesheets
```

## File Naming Conventions

- **Components**: kebab-case (e.g., `navbar.tsx`, `lets-talk-button.tsx`)
- **Pages**: lowercase with hyphens for routes (e.g., `about/page.tsx`)
- **Utilities**: kebab-case (e.g., `utils.ts`)
- **Types**: PascalCase interfaces/types
- **Constants**: UPPER_SNAKE_CASE

## Import Patterns

- Use `@/` path alias for all internal imports
- Components from `@/components/`
- UI components from `@/components/ui/`
- Utilities from `@/lib/`
- Hooks from `@/hooks/`

## Component Structure

- All components use TypeScript with proper typing
- Client components marked with `'use client'` directive
- Consistent export pattern: named exports preferred
- Props interfaces defined inline or separately for complex components

## Styling Conventions

- Tailwind CSS classes for all styling
- Custom CSS variables defined in `globals.css`
- Responsive design with mobile-first approach
- Dark/light theme support via CSS variables
- Component variants managed with `class-variance-authority`

## Asset Organization

- Images stored in `public/images/` with descriptive names
- Stack/technology logos in `public/images/stack/`
- Profile and project images in root `public/images/`
- SVG icons preferred, with Lucide React for UI icons

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thbueno) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
