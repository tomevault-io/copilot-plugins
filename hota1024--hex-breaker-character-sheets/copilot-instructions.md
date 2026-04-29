## hex-breaker-character-sheets

> This project uses Next.js 15 App Router with the following patterns:


# Next.js App Router Patterns and Conventions

## App Router Structure

This project uses Next.js 15 App Router with the following patterns:

### File-based Routing

- `app/page.tsx` - Homepage route (`/`)
- `app/layout.tsx` - Root layout applied to all routes
- `app/globals.css` - Global styles imported in root layout
- Future routes: `app/[route]/page.tsx` for dynamic routes

### Layout Patterns

The root layout in [app/layout.tsx](mdc:app/layout.tsx) includes:

- Font optimization with Geist fonts
- Theme provider for dark/light mode
- Proper HTML structure with `suppressHydrationWarning`
- Global CSS imports

```typescript
// Layout component pattern
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body
        className={`${geistSans.variable} ${geistMono.variable} antialiased`}
      >
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

## Metadata and SEO

- Export `metadata` object from page components for static metadata
- Use `generateMetadata` function for dynamic metadata
- Include proper Open Graph and Twitter Card metadata
- Set appropriate page titles and descriptions

```typescript
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Character Sheet - Hex Breaker",
  description: "Create and manage your Hex Breaker character sheets",
};
```

## Server and Client Components

- Use Server Components by default (no "use client" directive)
- Add "use client" only when needed for:
  - Event handlers (onClick, onChange, etc.)
  - Browser-only APIs (localStorage, window, etc.)
  - React hooks (useState, useEffect, etc.)
  - Interactive components

### Server Component Benefits

- Better performance and SEO
- Reduced bundle size
- Direct database access (when applicable)
- Automatic code splitting

### Client Component Usage

```typescript
"use client";

import { useState } from "react";

export default function InteractiveComponent() {
  const [count, setCount] = useState(0);
  // Client-side logic here
}
```

## Data Fetching Patterns

- Use async Server Components for data fetching
- Implement proper loading states with `loading.tsx` files
- Handle errors with `error.tsx` files
- Use React Suspense for streaming UI

## Image Optimization

- Always use `next/image` component for images
- Set appropriate `width`, `height`, and `alt` attributes
- Use `priority` prop for above-the-fold images
- Leverage Next.js automatic image optimization

## Performance Optimization

- Enable Turbopack for faster development builds
- Use dynamic imports for code splitting when needed
- Implement proper caching strategies
- Optimize Core Web Vitals metrics

## Development Scripts

- `pnpm dev` - Start development server with Turbopack
- `pnpm build` - Build production bundle with Turbopack
- `pnpm start` - Start production server
- Always test builds locally before deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hota1024) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
