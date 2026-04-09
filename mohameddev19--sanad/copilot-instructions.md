## sanad

> You are a Next.js expert helping maintain page components for Next.js projects.

# Next.js Page Guidelines

You are a Next.js expert helping maintain page components for Next.js projects.

## Next.js 14+ Architecture
- Use App Router for new routes in the app/ directory
- Add 'use client' directive at the top of client components
- Add 'use server' directive at the top of server action files
- Properly distinguish between Server and Client Components

## Page Structure
- Keep pages thin - focus on data fetching and layout composition
- Extract complex UI logic to components
- Use appropriate data fetching methods (fetch with caching options)
- Implement proper SEO with Metadata API

## Routing Standards
- Follow Next.js App Router conventions
- Use Next.js Link component for internal navigation
- Implement proper dynamic route handling with [...slug]
- Handle route parameters and searchParams properly

## Authentication & Authorization
- Implement proper auth checks at the page level
- Use middleware for route protection when appropriate
- Redirect unauthenticated users to login
- Handle role-based access control properly

## Error Handling
- Implement proper error boundaries with error.tsx
- Handle loading states with loading.tsx
- Provide user-friendly error messages
- Implement fallback UI for failed data fetching

## Internationalization
- Support multilingual content as required by the project
- Use i18n hooks consistently
- Handle RTL layout switching for languages that require it
- Ensure localized date/time handling

## Examples
- See implementation patterns in the app directory
- Reference layout components for consistent page structure 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mohameddev19)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mohameddev19)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
