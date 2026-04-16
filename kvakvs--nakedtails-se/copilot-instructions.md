## nakedtails-se

> This is a Typescript application using Next.js (Create Next App) and React.

This is a Typescript application using Next.js (Create Next App) and React.

# Next.js Best Practices

## Project Structure

- Use the App Router directory structure
- Place components in `app` directory for route-specific components
- Place shared components in `components` directory
- Place utilities and helpers in `lib` directory
- Use lowercase with dashes for directories (e.g., `components/auth-wizard`)

## Components

- Mark client components explicitly with 'use client'
- Use dynamic loading for non-critical components
- Implement proper error boundaries
- Place static content and interfaces at file end

## Performance

- Minimize use of 'useEffect' and 'setState'
- Use dynamic loading for non-critical components
- Implement proper caching strategies

## Routing

- Use the App Router conventions
- Implement proper loading and error states for routes
- Use dynamic routes appropriately
- Handle parallel routes when needed

## Forms and Validation

- Use Zod for form validation
- Implement proper server-side validation
- Handle form errors appropriately
- Show loading states during form submission

## State Management

- Minimize client-side state
- Use React Context sparingly
- Prefer server state when possible
- Implement proper loading states

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kvakvs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
