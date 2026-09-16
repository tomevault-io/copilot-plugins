## product-analytic-dashboard

> This is a Next.js 15.3.3 application using App Router and Prisma ORM. The project structure follows modern Next.js best practices with server components and TypeScript.

## Project Overview

This is a Next.js 15.3.3 application using App Router and Prisma ORM. The project structure follows modern Next.js best practices with server components and TypeScript.

## Architecture & Tech Stack

### Core Dependencies

- **Next.js**: 15.3.3 (App Router)
- **React**: 19.0.0
- **TypeScript**: 5.x
- **Database**: PostgreSQL with Prisma 6.9.0 ORM
- **Styling**: Tailwind CSS 4.x with custom CSS variables
- **UI Components**: Radix UI primitives with shadcn/ui patterns
- **Package Manager**: Npm

## Development Rules & Guidelines

### NEVER USE

- `middleware.ts` - Authentication and routing should be handled in components/pages
- `"use client;"` in `layout.tsx` or `page.tsx` files - Only use in actual components
- `npm run build` unless specifically requested - Assume `npm run dev` is running

### Always Use

- **Package Manager**: `NPM` for all package operations
- **Server Components**: Default to Server Components for data fetching
- **App Router**: Use Next.js 15 App Router patterns
- **TypeScript**: Strict typing for all code

### Route Parameters

When a route uses params, always follow this pattern:

```typescript
// CORRECT
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  // use id here
}

// WRONG
export default async function Page({ params }: { params: { id: string } }) {
  // direct usage without awaiting
}
```

### Styling Guidelines

- **Primary Styling**: All styles in `globals.css` using Tailwind CSS
- **Design System**: Uses custom CSS variables for theming (light/dark mode support)
- **UI Components**: Use existing shadcn/ui components from `components/ui/`:
  - `badge`, `button`, `calendar`, `card`, `dialog`, `form`, `input`, `label`, `popover`, `select`, `sonner`
- **Loading States**: Use Skeleton components from shadcn (check if specific skeleton exists first)
- **Icons**: Use Lucide React For icons.

### Database & Prisma

- **ORM**: Prisma with PostgreSQL
- **Client Location**: Generated to `app/generated/prisma` (don't change this.)

### Error Handling & UX

- **Error Boundaries**: Use shadcn `toast` for error notifications
- **Loading States**: Implement skeleton loading components
- **Form Validation**: Use `react-hook-form` with proper validation
- **Toast Notifications**: `Toaster` component from sonner is configured in root layout

### Component Patterns

- **Server Components**: Default for data fetching and static content
- **Client Components**: Only when needed for interactivity (`"use client;"`)
- **Providers**: Session provider wraps entire app in root layout
- **Conditional Rendering**: Based on user roles and subscription status

### Code Quality Standards

- **No Comments**: Code should be self-documenting
- **TypeScript Strict**: All code must pass TypeScript strict mode
- **Error Handling**: Proper try-catch blocks in API routes
- **Validation**: Validate all inputs, especially in API routes

### Testing Strategy

- **Route Testing**: Assume `npm run dev` is running, test routes directly
- **Component Testing**: Focus on functionality over implementation
- **Database Testing**: Use separate test database if needed

This codebase follows modern Next.js patterns with server-first architecture.

---
> Source: [DesHPJ/Product_analytic-dashboard](https://github.com/DesHPJ/Product_analytic-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
