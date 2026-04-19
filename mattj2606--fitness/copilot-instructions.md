## fitness

> This project is a personal, self-hosted fitness intelligence app designed for daily use on mobile (iPhone) via a mobile-first web app. The app focuses on fast daily check-ins, structured workout logging, muscle-level analytics, and simple, explainable recommendations.

# Fitness App - Cursor Rules

This project is a personal, self-hosted fitness intelligence app designed for daily use on mobile (iPhone) via a mobile-first web app. The app focuses on fast daily check-ins, structured workout logging, muscle-level analytics, and simple, explainable recommendations.

## Project Overview

- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Database**: PostgreSQL with Prisma ORM
- **Platform**: Mobile-first PWA
- **Pattern**: React Server Components + API Routes
- **Focus**: Data ingestion interface - optimized for fast, mobile data entry

**Key URLs**:
- GitHub Repository: (to be added)

## Project Structure

```
app/
  (routes)/          # Route groups for organization
  api/               # API routes (route.ts files)
  components/        # Reusable components
    ui/              # Base UI components
    checkin/         # Feature-specific components
    workout/
    analytics/
  lib/               # Utilities and shared code
    db/              # Database utilities (Prisma)
    services/        # Business logic
    utils/           # Helper functions
    types/           # TypeScript types
```

## Next.js App Router Best Practices

1. **Use Server Components by default** - Only use Client Components when necessary for interactivity
2. **Implement the file-based routing system** - Use `app/` directory structure
3. **Use `layout.tsx` for shared layouts** - Nest layouts for route groups
4. **Implement `loading.tsx` for loading states** - Automatic Suspense boundaries
5. **Use `error.tsx` for error handling** - Route-level error boundaries
6. **Utilize route handlers (`route.ts`) for API routes** - Not Pages Router API routes
7. **Use metadata API for SEO** - Export metadata from page/layout files
8. **Use Next.js Image component** - Optimized image loading with WebP support
9. **Use environment variables** - `.env.local` for configuration

## TypeScript Best Practices

- **Use TypeScript for all code** - Prefer interfaces over types
- **Avoid enums** - Use maps or const objects instead
- **Use functional components with TypeScript interfaces**
- **Avoid `any`** - Use `unknown` if needed with proper type guards
- **Use Prisma-generated types** - Leverage type safety from database schema
- **Use Zod for runtime validation** - Form validation and API boundary validation

## Component Patterns

### Component Definition Syntax

```tsx
// Server Component (default)
export const ComponentName = () => {
  // Component logic
};

// Client Component (when needed)
'use client'

interface ComponentNameProps {
  // Props definition
}

export const ComponentName = ({ prop1, prop2 }: ComponentNameProps) => {
  // Component logic
};

// Page components use default exports
const Page = () => {
  // Page component logic
};

export default Page;
```

### File Organization Structure

For component files, structure in this order:
1. Exported component
2. Subcomponents
3. Helper functions
4. Static content
5. Types/interfaces

### Naming Conventions

- **Components**: PascalCase (`CheckinForm.tsx`)
- **Utilities**: camelCase (`formatDate.ts`)
- **Constants**: UPPER_SNAKE_CASE (`API_ENDPOINTS.ts`)
- **Types/Interfaces**: PascalCase (`WorkoutData.ts`)
- **Directories**: lowercase with dashes (`components/auth-wizard/`)
- **API routes**: `route.ts` (lowercase)
- **Favor named exports** for components

## Code Style and Structure

- Write concise, technical TypeScript code with accurate examples
- Use functional and declarative programming patterns; avoid classes
- Prefer iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`)
- Use the "function" keyword for pure functions
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements
- Use declarative JSX
- Minimize AI-generated comments; use clearly named variables and functions instead

## Error Handling and Validation

- **Prioritize error handling**: Handle errors and edge cases early
- **Use early returns and guard clauses**
- **Implement proper error logging** with user-friendly messages
- **Use Zod for form validation** - Type-safe validation schemas
- **Model expected errors as return values** in Server Actions
- **Use error boundaries** (`error.tsx`) for unexpected errors
- **Never expose sensitive error details** to client
- **Use consistent error response format** in API routes

## UI and Styling

- **Use Tailwind CSS exclusively** for styling - Avoid inline styles
- **Mobile-first approach**: Base styles for mobile, then `md:`, `lg:` breakpoints
- **Ensure touch targets are at least 44x44px** for mobile
- **Use consistent spacing scale** from Tailwind
- **Test on mobile viewport sizes** regularly
- **Implement responsive design** with Tailwind CSS breakpoints
- **Use semantic HTML** for accessibility
- **Include ARIA labels** where needed
- **Ensure keyboard navigation** works
- **Maintain proper heading hierarchy**
- **Provide alt text for images**
- **Ensure sufficient color contrast**

## Performance Optimization

- **Minimize `useEffect` and `setState`** - Favor React Server Components (RSC)
- **Wrap client components in Suspense** with fallback
- **Use dynamic loading** for non-critical components
- **Optimize images**: Use WebP format, include size data, implement lazy loading
- **Use Next.js built-in caching** - `fetch` with `next: { revalidate }` option
- **Optimize database queries** - Avoid N+1 queries, use Prisma efficiently
- **Cache API responses** when appropriate
- **Implement proper loading states** - Use `loading.tsx` files

## Data Fetching Patterns

### Server Components (Preferred)

```tsx
async function getData() {
  const res = await fetch('https://api.example.com/data', { 
    next: { revalidate: 3600 } 
  })
  if (!res.ok) throw new Error('Failed to fetch data')
  return res.json()
}

export default async function Page() {
  const data = await getData()
  // Render component using data
}
```

### API Routes (route.ts)

```tsx
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  // Handle GET request
  return NextResponse.json({ data: 'example' })
}

export async function POST(request: NextRequest) {
  // Handle POST request
  const body = await request.json()
  // Validate with Zod
  return NextResponse.json({ success: true })
}
```

## Database and Prisma Patterns

- **Always use Prisma Client** for database access - Never write raw SQL (unless absolutely necessary)
- **Use transactions** (`prisma.$transaction()`) for multi-step operations
- **Handle database errors gracefully** - Proper try-catch blocks
- **Index frequently queried fields** - Optimize for common queries
- **Use Prisma-generated types** - Leverage type safety
- **Use parameterized queries** - Prisma handles this automatically
- **Design for time-series queries** - Workout history, trends, analytics

## API Route Patterns

- **Use Next.js route handlers** (`route.ts` files in `app/api/`)
- **Validate all inputs with Zod schemas** - Type-safe validation
- **Return consistent error response format**
- **Use proper HTTP status codes** (200, 201, 400, 401, 404, 500, etc.)
- **Handle errors with try-catch blocks**
- **Log errors appropriately** - For debugging
- **Always authenticate requests** - Except public routes
- **Use HTTP-only cookies** for sessions
- **Protect API routes** with authentication middleware

## Security

- **Validate all user inputs** - Use Zod schemas
- **Sanitize data** before database operations (Prisma handles SQL injection)
- **Implement proper authentication checks** - Session-based auth for single user
- **Use HTTP-only cookies** for sessions - XSS protection
- **Protect API routes** with authentication middleware
- **Never expose sensitive error details** to client
- **Use environment variables** for secrets and configuration

## Mobile-First Development

- **Always consider mobile viewport first** - Mobile-first CSS approach
- **Test interactions with touch in mind** - Large touch targets
- **Ensure forms are easy to complete on mobile** - Minimal typing required
- **Minimize typing requirements** - Use dropdowns, buttons, sliders
- **Use large touch targets** - Minimum 44x44px
- **Consider one-handed usage** - Place primary actions in thumb zone
- **Optimize for fast data entry** - Primary focus of this app

## Data Ingestion Focus

This app is optimized for **fast data entry** - check-ins, workout logging, quick updates:

- **Optimize for fast data entry** - Minimize form fields
- **Use smart defaults** - Pre-fill when possible
- **Auto-save when possible** - Reduce data loss risk
- **Provide clear validation feedback** - Real-time validation
- **Support offline entry** - IndexedDB + background sync
- **Minimize text input** - Use buttons, sliders, selectors
- **Fast form completion** - Vertical flow, clear progression

## PWA Considerations

- **Service Worker** for offline workout logging
- **IndexedDB** for local storage during offline periods
- **Background sync** for uploading when connection restored
- **PWA manifest** - App name, icons, theme colors, standalone mode
- **Offline-first approach** - Critical for gym environments

## React Server Components Guidelines

- **Favor server components** and Next.js SSR
- **Use client components only for**:
  - Web API access (localStorage, window, etc.)
  - Interactivity (onClick, onChange, etc.)
  - Browser-only features
- **Avoid client components for**:
  - Data fetching (use Server Components)
  - State management (use Server Components + URL state)
- **Mark client components** with `'use client'` directive at top of file

## Code Organization

- **Atomic design principles**: atoms → molecules → organisms → templates → pages
- **Place components** in `components/` directory
- **Group related components** in subdirectories (`components/checkin/`, `components/workout/`)
- **Co-locate** component files with related utilities when appropriate
- **Use index files** for clean imports when needed

## Metadata and SEO

```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Page Title',
  description: 'Page description',
}
```

## Error Boundary Pattern

```tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

## Key Conventions

- Use proper URL search parameter state management
- Optimize Web Vitals (LCP, CLS, FID)
- Limit `'use client'` usage - Prefer Server Components
- Follow Next.js 14 App Router docs for Data Fetching, Rendering, and Routing
- Use functional components exclusively
- Keep components small and focused
- Use composition over inheritance
- Extract reusable logic to custom hooks when needed

## Development Guidelines

- Use TypeScript for type safety
- Follow coding standards defined in ESLint configuration
- Ensure all components are responsive and accessible
- Use Tailwind CSS for styling, adhering to defined patterns
- Always validate user inputs and handle errors gracefully
- Use existing components and pages as reference for new components
- Minimize the use of AI-generated comments
- Use clearly named variables and functions

## AI Interaction Guidelines

- When generating code, prioritize TypeScript and React best practices
- Ensure that any new components are reusable and follow existing design patterns
- Always validate user inputs and handle errors gracefully
- Use existing components and pages as reference for new components and pages
- Prefer existing patterns over new approaches
- Ask for clarification if requirements are unclear
- Suggest improvements if a better pattern exists
- Maintain consistency with existing codebase

## Testing Considerations

- Write testable code (pure functions where possible)
- Extract business logic from components
- Use dependency injection for testability
- Document complex logic with comments
- Consider test cases when implementing features

## Documentation

- Comment complex business logic
- Document API endpoints (inline comments or JSDoc)
- Use JSDoc for public functions
- Keep README updated
- Document architecture decisions

## Important Scripts

- `dev`: Starts the development server
- `build`: Builds the application for production
- `start`: Starts the production server
- `lint`: Runs ESLint
- `type-check`: Runs TypeScript type checking

## Additional Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [React Documentation](https://react.dev/)
- [Prisma Documentation](https://www.prisma.io/docs)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattj2606) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
