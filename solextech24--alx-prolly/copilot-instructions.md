## alx-prolly

> This is a Next.js 15 polling application that uses Supabase for authentication and database, shadcn/ui for components, and follows specific architectural patterns.

# ALX Polling App - Cursor Rules

## Project Overview
This is a Next.js 15 polling application that uses Supabase for authentication and database, shadcn/ui for components, and follows specific architectural patterns.

## Core Technology Stack
- **Framework**: Next.js 15 (App Router)
- **Database & Auth**: Supabase
- **UI Components**: shadcn/ui with Tailwind CSS
- **Forms**: react-hook-form with zod validation
- **Icons**: Lucide React
- **State**: Server Components + local state for forms

## File Structure & Naming Conventions

### Directory Organization
- `/app` - Next.js App Router pages and API routes
- `/app/(auth)` - Authentication-related pages (login, register)
- `/app/(dashboard)` - Protected dashboard pages
- `/app/api` - API route handlers
- `/app/polls` - Poll-related pages
- `/components` - Reusable UI components
- `/components/ui` - shadcn/ui components
- `/components/forms` - Form components
- `/components/layout` - Layout and navigation components
- `/components/polls` - Poll-specific components
- `/lib` - Utility functions and configurations
- `/lib/utils/supabase` - Supabase client configuration

### Naming Conventions
- **Components**: PascalCase (e.g., `CreatePollForm.tsx`, `PollView.tsx`)
- **Files**: kebab-case for pages, PascalCase for components
- **Functions**: camelCase (e.g., `createPoll`, `handleVote`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `NEXT_PUBLIC_SUPABASE_URL`)

## Component Patterns

### Form Components
- Use `'use client'` directive for interactive forms
- Implement form validation with zod schemas
- Use shadcn/ui form components (`Input`, `Textarea`, `Select`, `Button`)
- Handle loading states with `useState` for `isLoading`
- Implement error handling with try-catch blocks
- Use `useRouter` for navigation after form submission

### API Route Patterns
- Use Next.js API routes (`/app/api/`)
- Implement proper error handling with try-catch
- Return consistent JSON responses with status codes
- Use Supabase client for database operations
- Validate request body data before processing

### Authentication & Authorization
- Use Supabase authentication helpers
- Implement protected routes with auth middleware
- Use environment variables for Supabase configuration
- Never hardcode API keys or secrets

## Code Style & Best Practices

### Imports & Dependencies
- Use absolute imports with `@/` alias (configured in `tsconfig.json`)
- Import shadcn/ui components from `@/components/ui`
- Import utility functions from `@/lib/utils`
- Use named imports for better tree-shaking

### State Management
- Prefer Server Components for data fetching
- Use `useState` for local component state (forms, UI interactions)
- Avoid client-side data fetching in page components
- Use Server Actions for data mutations when possible

### Error Handling
- Implement comprehensive error handling in API routes
- Use user-friendly error messages
- Log errors for debugging
- Provide fallback UI for error states

### Styling
- Use Tailwind CSS classes consistently
- Leverage shadcn/ui component variants
- Follow the established design system
- Use CSS variables for theming when needed

## Database & API Patterns

### Supabase Usage
- Use environment variables for Supabase configuration
- Implement proper error handling for database operations
- Use service role key for server-side operations
- Use anon key for client-side operations

### Data Validation
- Validate all user inputs on both client and server
- Use zod schemas for type safety
- Sanitize data before database operations
- Implement proper input sanitization

## Performance & Optimization

### Next.js Best Practices
- Use Server Components for static content
- Implement proper loading states
- Use Next.js Image component for images
- Optimize bundle size with proper imports

### Component Optimization
- Lazy load non-critical components
- Implement proper memoization when needed
- Use React.memo for expensive components
- Avoid unnecessary re-renders

## Security Considerations

### Environment Variables
- Never commit `.env.local` files
- Use `NEXT_PUBLIC_` prefix only for client-side variables
- Keep service role keys server-side only
- Validate environment variables at startup

### Input Validation
- Sanitize all user inputs
- Implement rate limiting for API endpoints
- Validate file uploads and sizes
- Use CSRF protection where applicable

## Testing & Development

### Development Workflow
- Use `npm run dev` for development
- Use `npm run build` to check for build errors
- Use `npm run lint` for code quality
- Use Prisma commands for database operations

### Code Quality
- Follow TypeScript strict mode
- Use ESLint for code linting
- Implement proper TypeScript types
- Use consistent code formatting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/solextech24) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
