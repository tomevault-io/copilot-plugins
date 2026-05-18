## helios-portal

> You are an expert in TypeScript, Next.js 15 App Router, React, Sass CSS

You are an expert in TypeScript, Next.js 15 App Router, React, Sass CSS

Key Principles

- Write concise, technical TypeScript code with accurate examples.
- Use functional and declarative programming patterns; avoid classes.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Structure files: exported component, subcomponents, helpers, static content, types.

Naming Conventions

- Use lowercase with dashes for directories (e.g., components/auth-wizard).
- Favor named exports for components.

TypeScript Usage

- Use TypeScript for all code; prefer interfaces over types.
- Avoid enums; use maps instead.
- Use functional components with TypeScript interfaces.

Syntax and Formatting

- Use the "function" keyword for pure functions.
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
- Use declarative JSX.

UI and Styling

- Use SASS with custom CSS variables, mixins for reusability, and BEM for class naming.
- Import CSS Modules with `import s from './component.module.scss'` and use classes as `s.className`.
- Keep styles scoped to components using CSS Modules pattern.
- Use `clsx` for combining multiple classes or conditional class application: `className={clsx(s.base, isActive && s.active, variant && s[variant])}`.
- ALWAYS refer to the global style files for variables and mixins:
  - Use color variables from `@/styles/_colors.scss`
  - Use size and layout variables from `@/styles/_variables.scss`
  - Use mixins from `@/styles/_mixins.scss`
  - Use easing/animation variables from `@/styles/_easing.scss`

Performance Optimization

- Minimize 'use client', 'useEffect', and 'setState'; favor React Server Components (RSC).
- Wrap client components in Suspense with fallback.
- Use dynamic loading for non-critical components.
- Optimize images: use WebP format, include size data, implement lazy loading.

Key Conventions

- Use 'nuqs' for URL search parameter state management.
- Optimize Web Vitals (LCP, CLS, FID).
- ALWAYS use environment variables through `@/env` (via @t3-oss/env-nextjs):
  - DO: `import { env } from '@/env'; ... env.NEXT_PUBLIC_API_URL`
  - DON'T: `process.env.NEXT_PUBLIC_API_URL`
  - This ensures type safety and validation through Zod
- Limit 'use client':
  - Favor server components and Next.js SSR.
  - Use only for Web API access in small components.
  - Avoid for data fetching or state management.
- ALWAYS use the Icon component for displaying icons:
  - DO: `import { Icon } from '@/components/icon'; ... <Icon icon="user" />`
  - DON'T: `import { Icon } from '@iconify/react'; ... <Icon icon="mdi:user" />`
- ALWAYS use MetadataSeo for page metadata:
  - DO: `import { MetadataSeo } from "@/lib/metadata"; ... export const metadata = MetadataSeo({ title: "Page Title", description: "Page description" })`
  - DON'T: Create metadata objects manually
- ALWAYS use the custom Link component for navigation:
  - DO: `import { Link } from '@/components/link'; ... <Link href="/page">Text</Link>`
  - DON'T: `import Link from 'next/link'; ... <Link href="/page">Text</Link>`
  - This ensures consistent handling of external links, anchor links, and click events

Follow Next.js docs for Data Fetching, Rendering, and Routing.

---
> Source: [helios-network/helios-portal](https://github.com/helios-network/helios-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
