## tommcclean-me

> - **Next.js 16** with App Router

# Cursor Rules for tommcclean.me

## Tech Stack
- **Next.js 16** with App Router
- **TypeScript** for type safety
- **Redux Toolkit** for state management
- **Tailwind CSS** for styling
- **Static Site Generation (SSG)** - configured for static export

## Core Principles

### 1. Redux-Driven Content
- **ALL static text and images MUST be driven by Redux state**, never hardcoded in components
- Each section has its own Redux slice in `lib/slices/`
- Use `useAppSelector` and `useAppDispatch` from `@/lib/hooks` to interact with Redux
- When adding new content, create or update the appropriate slice, not the component

### 2. Component Structure
- Use default exports for components
- Add `'use client'` directive when component needs interactivity (useState, event handlers, etc.)
- Components should be in `components/` directory
- Use TypeScript interfaces/types for props and data structures

### 3. Redux Slices
- Each slice should define its state interface
- Use `createSlice` from Redux Toolkit
- Export the slice reducer and any action creators
- Keep initial state in the slice file
- Slice files should be in `lib/slices/` directory

### 4. Styling Conventions
- Use **Tailwind CSS** utility classes exclusively
- **Mobile-first responsive design**: Start with mobile styles, then add `sm:`, `md:`, `lg:` breakpoints
- Support dark mode: Use `dark:` variants for dark mode styles
- Color scheme: Use zinc color palette (zinc-100, zinc-900, etc.)
- Sections should have: `border-t-2 border-zinc-300 bg-white dark:border-zinc-700 dark:bg-black`
- Container: `mx-auto max-w-6xl px-6 py-20`
- Use consistent spacing: `space-y-8` for vertical spacing between items

### 5. Images
- Store images in `public/images/` directory
- Use Next.js `Image` component from `next/image` for optimized images
- Reference images with paths starting with `/images/` (e.g., `/images/avatar.jpg`)
- Always provide `alt` text for accessibility
- Use `object-contain` for logos and images that should maintain aspect ratio
- Set appropriate `width` and `height` props

### 6. Responsive Design Patterns
- Mobile: Stack items vertically, center content (`flex-col`, `items-center`, `text-center`)
- Desktop: Horizontal layout (`sm:flex-row`, `sm:items-start`, `sm:text-left`)
- Logo positioning: On mobile, logos should be above text; on desktop, to the left
- Example pattern:
  ```tsx
  <div className="flex flex-col items-center gap-4 sm:flex-row sm:items-start">
    <div className="flex-shrink-0 sm:mr-4">{/* Logo */}</div>
    <div className="flex-1 text-center sm:text-left">{/* Content */}</div>
  </div>
  ```

### 7. TypeScript
- Use strict TypeScript typing
- Define interfaces for all data structures
- Use type imports: `import type { ... }` for types only
- Export types from slice files for reuse in components

### 8. File Organization
```
app/                    # Next.js app directory
  ├── layout.tsx       # Root layout with Redux provider
  ├── page.tsx         # Home page
  └── globals.css      # Global styles
components/            # React components
lib/                   # Utilities and Redux setup
  ├── store.ts         # Redux store configuration
  ├── hooks.ts         # Typed Redux hooks
  └── slices/          # Redux slices
public/                # Static assets
  └── images/          # Image files
```

### 9. Static Site Generation
- Site is configured for static export (`output: 'export'` in `next.config.ts`)
- Images are unoptimized for static export (`images: { unoptimized: true }`)
- Build output goes to `out/` directory
- No server-side features should be used (no API routes, server components with dynamic data, etc.)

### 10. ESLint Configuration
- **ESLint** is configured with Next.js recommended rules (`eslint-config-next`)
- Uses both `core-web-vitals` and `typescript` configs from Next.js
- Run linting with: `npm run lint`
- ESLint is configured in `eslint.config.mjs`

#### Popular ESLint Rules to Follow

**React/Next.js Rules:**
- `react-hooks/rules-of-hooks`: Only call hooks at the top level
- `react-hooks/exhaustive-deps`: Include all dependencies in useEffect/useMemo/useCallback
- `@next/next/no-img-element`: Use Next.js `Image` component instead of `<img>`
- `@next/next/no-html-link-for-pages`: Use Next.js `Link` component for internal links
- `react/no-unescaped-entities`: Escape special characters in JSX text
- `react/jsx-key`: Provide keys for list items

**TypeScript Rules:**
- `@typescript-eslint/no-unused-vars`: Remove unused variables and imports
- `@typescript-eslint/no-explicit-any`: Avoid `any` type, use proper types
- `@typescript-eslint/explicit-function-return-type`: Prefer explicit return types for functions
- `@typescript-eslint/no-non-null-assertion`: Avoid non-null assertions (`!`), use proper null checks

**General Best Practices:**
- Remove unused imports and variables
- Use `const` for variables that don't change, `let` only when reassignment is needed
- Prefer arrow functions for callbacks
- Use template literals instead of string concatenation
- Avoid `console.log` in production code (use proper logging)
- Use meaningful variable names (avoid single letters except for loop counters)
- Keep functions small and focused (single responsibility)

**Accessibility Rules:**
- Always provide `alt` text for images
- Use semantic HTML elements (`<button>` instead of `<div>` for buttons)
- Ensure interactive elements are keyboard accessible

**Performance Rules:**
- Avoid creating objects/functions in render methods
- Use `useMemo` and `useCallback` appropriately for expensive operations
- Optimize images with Next.js `Image` component

### 11. Code Quality
- Keep components focused and single-purpose
- Extract reusable logic into hooks or utilities
- Use meaningful variable and function names
- Add comments for complex logic
- Remove unused code and imports
- Follow ESLint rules and fix warnings/errors

### 12. Common Patterns

#### Redux Hook Usage
```typescript
import { useAppSelector } from '@/lib/hooks';

const experience = useAppSelector((state) => state.experience);
```

#### Section Component Structure
```typescript
'use client'; // If needed

export default function SectionName() {
  const data = useAppSelector((state) => state.sliceName);
  
  return (
    <section id="section-id" className="border-t-2 border-zinc-300 bg-white dark:border-zinc-700 dark:bg-black">
      <div className="mx-auto max-w-6xl px-6 py-20">
        {/* Content */}
      </div>
    </section>
  );
}
```

#### Image Usage
```typescript
import Image from 'next/image';

<Image
  src="/images/example.jpg"
  alt="Description"
  width={72}
  height={72}
  className="h-full w-full rounded-lg object-contain"
/>
```

## When Making Changes

1. **Adding new content**: Update the appropriate Redux slice, not the component
2. **Adding new sections**: Create a new slice, add it to `lib/store.ts`, create component, add to `app/page.tsx`
3. **Styling changes**: Use Tailwind classes, maintain mobile-first approach, support dark mode
4. **Responsive fixes**: Ensure both mobile and desktop layouts work correctly
5. **Type safety**: Update TypeScript interfaces when changing data structures
6. **Code quality**: Run `npm run lint` before committing, fix all ESLint errors and warnings

## Deployment
- Build command: `npm run build`
- Output directory: `out/`
- Node version: 20 (specified in `.nvmrc` and `package.json`)
- Configured for Cloudflare Pages deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tommcclean) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
