## facecoin

> - Start with prompt analysis and pseudocode planning

# Code Implementation

- Start with prompt analysis and pseudocode planning
- Use comments for logic explanation, not code description
- Provide complete, functional code with minimal placeholders
- Prioritize readability over optimization
- Reference files to be modified
- Challenge assumptions and show complete thought process

# TypeScript & React Guidelines

- Use TypeScript's advanced features (generics, utility types, mapped types, tagged unions, branded types)
- Write explicit, declarative code with functional patterns
- Use descriptive names with auxiliary verbs (isLoading, hasError)
- Follow SOLID principles and DRY
- Avoid nesting and reduce bracket usage
- Use types instead of interfaces
- Avoid enums and any types

# Project Structure

- Use Next.js 15 App Router conventions:
  - page.tsx for pages
  - layout.tsx for layouts
  - loading.tsx for suspense boundaries
  - error.tsx for error handling
  - route.ts for API routes
  - always use Link from next/link for navigation
- Separate business logic from UI
- Use index files for clean imports

# React & Next.js Specifics

- Use Next.js 15 App Router with React Server Components
- Implement Server Actions for form submissions
- Use Client Components sparingly with 'use client' directive
- Use Shadcn UI components from components/shadcn/\*
- Implement Suspense boundaries with loading.tsx
- Use react-query for client-side data fetching
- Mobile-first with Tailwind CSS
- Ensure accessibility with ARIA attributes
- Leverage React Server Components for:
  - Data fetching
  - Database queries
  - API calls
  - Static content

# Security

- Sanitize frontend inputs
- Use environment variables for sensitive data
- Enforce HTTPS and CORS
- Implement middleware for authentication
- Use Next.js middleware for route protection

# Core Dependencies

- next, react: Core framework (App Router)
- @tanstack/react-query: Client-side data fetching
- viem: Ethereum interactions
- square: Payment processing
- @vercel/postgres: Main database
- @privy-io/\*: Authentication

# Performance Optimization

- Use React Suspense for code splitting
- Implement route segments for code splitting
- Use generateMetadata for SEO
- Implement proper caching strategies:
  - fetch cache options
  - revalidate paths
  - on-demand revalidation
- Use Next.js Image component for optimized images
- Implement proper loading states with Suspense

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0xmetropolis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
