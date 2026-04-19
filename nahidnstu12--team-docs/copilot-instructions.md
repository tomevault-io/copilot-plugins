## team-docs

> - Use arrow functions for component definitions and callbacks


# General Code Style & Formatting

- Use arrow functions for component definitions and callbacks
- Use Bun for package management
- Deploy on Vercel
- Follow Laravel-like class system (e.g., `BaseService.js`, `UserService.js`)
- Follow ESLint and Prettier configurations

# Project Structure & Architecture

- Follow Next.js V15+ patterns and using App Router.
- Use server components by default, client components only when necessary
- Directory structure:
- if specific hook, then create the hook near the directory (like app/user/hook/) otherwise at src/hooks/
- if a page needed to be split into components, then put it near the directory (like app/user/components/) otherwise src/components/
- if client component need to fetch something from server we create the server function near the route (like app/user/actions/) and use react useTransition hook to fetch data & show loading state
- if a feature need zustand store, create it near the route (like app/user/store)
- always try to create server function instead of route handler.
- using authjs jwt with posgres database with prisma orm with credentials login only.
- use nextjs dynamic to load the client component lazily with loading sapinner and add no ssr option.
- each page should contains most nextjs common file conventions (like not-found, error, unauthorized etc)
- to protect route, utilize middleware, server function & server component
- handle edge cases on server component

# Styling & UI

- Use Tailwind CSS V4 for styling.
- Use Shadcn UI for components.
- Use Lucid React Icon.
- use Framer Motion for animation.
- Follow mobile-first responsive design
- Use CSS variables for theming

# Data Fetching & Forms

- Use React Hook Form for form handling.
- Use Zod for validation.
- use useActionState hook to manage form with formAction.
- use useTransition for fetch & loading state.
- Implement proper error boundaries
- Use server actions for form submissions
- Implement optimistic updates for better UX

# State Management & Logic

- Use React Context for:
  - Theme
  - Authentication state
  - App-wide UI state (modals, toasts)
  - Read-only global state
- Use Zustand for:
  - Complex client-side state
  - Global state that needs persistence
  - State shared across many components

# Backend & Database

- Use Prisma for database access.
- we are using postgres db.

# Performance

- Use `next/dynamic` for lazy loading components
- Implement proper loading states with `Suspense`
- Use `unstable_cache` for data caching
- Optimize images with `next/image`
- Use `React.memo` for expensive components
- Implement proper code splitting
- Monitor performance using Web Vitals

# Error Handling

- Use error boundaries for client-side errors
- Implement proper error pages:
  - `error.js` - Client-side errors
  - `not-found.js` - 404 errors
  - `global-error.js` - Root error boundary
- use custom Logger class to print out errors
- Provide user-friendly error messages

# Documentation

- Document component props using JSDoc
- Add comments for complex logic
- Keep README.md updated

# Security

- Implement proper input validation
- Sanitize all user inputs
- Protect against XSS, CSRF, and other common vulnerabilities
- Use environment variables for sensitive data
- Implement rate limiting
- Use HTTPS in production

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nahidnstu12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
