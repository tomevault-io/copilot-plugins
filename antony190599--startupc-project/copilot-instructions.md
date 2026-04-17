## startupc-project

> This is a Next.js project using:

# Cursor Rules for Next.js Project

## Project Overview
This is a Next.js project using:
- Next.js 14+ with App Router
- TypeScript
- Tailwind CSS for styling
- Shadcn/ui for UI components
- Next Auth for authentication
- Prisma for database management

## Code Style & Standards

### TypeScript
- Use strict TypeScript configuration
- Prefer interfaces over types for object shapes
- Use proper type annotations for all function parameters and return types
- Avoid `any` type - use `unknown` or proper typing instead
- Use generic types when appropriate

### Next.js Best Practices
- Use App Router (app/ directory) instead of Pages Router
- Implement proper error boundaries with error.tsx
- Use loading.tsx for loading states
- Follow Next.js file conventions (page.tsx, layout.tsx, etc.)
- Use Server Components by default, Client Components only when necessary
- Implement proper metadata in layout.tsx and page.tsx files

### Component Structure
- Use PascalCase for component names
- Place components in src/components/
- Separate UI components (src/components/ui/) from feature components
- Use index.ts files for clean imports
- Implement proper prop interfaces for all components

### Tailwind CSS Guidelines
- Use Tailwind utility classes for styling
- Follow mobile-first responsive design
- Use semantic class names and avoid arbitrary values when possible
- Group related classes logically (layout, spacing, colors, etc.)
- Use @apply sparingly, prefer utility classes in JSX

### Shadcn/ui Integration
- Use shadcn/ui components as the primary UI library
- Follow shadcn/ui patterns for component composition
- Customize components through CSS variables when needed
- Maintain consistency with shadcn/ui design system

### Authentication (Next Auth)
- Implement proper session management
- Use Next Auth providers appropriately
- Secure API routes with authentication middleware
- Handle authentication states properly in components
- Use proper error handling for auth failures

### Database (Prisma)
- Use Prisma Client for database operations
- Implement proper error handling for database queries
- Use transactions when multiple operations need to be atomic
- Follow Prisma naming conventions
- Use proper relationships and foreign keys
- Implement proper data validation before database operations

## File Organization

### Directory Structure
```
src/
├── app/                    # Next.js App Router pages
├── components/             # React components
│   ├── ui/                # Shadcn/ui components
│   └── [feature]/         # Feature-specific components
├── lib/                   # Utility functions and configurations
├── prisma/                # Prisma schema and migrations
├── auth/                  # Next Auth configuration
└── types/                 # TypeScript type definitions
```

### Naming Conventions
- Files: kebab-case (e.g., user-profile.tsx)
- Components: PascalCase (e.g., UserProfile)
- Functions: camelCase (e.g., getUserData)
- Constants: UPPER_SNAKE_CASE (e.g., API_ENDPOINTS)
- Database tables: snake_case (e.g., user_profiles)

## Development Guidelines

### Error Handling
- Implement proper try-catch blocks
- Use Next.js error boundaries
- Provide meaningful error messages
- Log errors appropriately
- Handle edge cases gracefully

### Performance
- Use Next.js Image component for images
- Implement proper loading states
- Use React.memo for expensive components
- Optimize database queries
- Implement proper caching strategies

### Security
- Validate all user inputs
- Use proper authentication checks
- Sanitize data before database operations
- Implement proper CORS policies
- Use environment variables for sensitive data

### Testing
- Write unit tests for utility functions
- Test component rendering and interactions
- Test API routes
- Use proper mocking for external dependencies

## Common Patterns

### API Routes
```typescript
// app/api/[route]/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  try {
    // Implementation
    return NextResponse.json({ data })
  } catch (error) {
    return NextResponse.json({ error: 'Message' }, { status: 500 })
  }
}
```

### Database Operations
```typescript
// lib/db.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

export async function getUserData(userId: string) {
  try {
    return await prisma.user.findUnique({
      where: { id: userId },
      include: { profile: true }
    })
  } catch (error) {
    console.error('Database error:', error)
    throw new Error('Failed to fetch user data')
  }
}
```

### Authentication
```typescript
// lib/auth.ts
import { getSession } from '@/lib/auth/utils';

export async function getCurrentUser() {
  const session = await getSession();
  return session?.user
}
```

## Environment Setup
- Use .env.local for local development
- Use .env.example for documentation
- Never commit sensitive data to version control
- Use proper environment variable validation

## Dependencies
- Keep dependencies up to date
- Use exact versions for critical dependencies
- Document any specific version requirements
- Use pnpm for package management

## Code Quality
- Use ESLint and Prettier
- Follow consistent formatting
- Write self-documenting code
- Use meaningful variable and function names
- Add comments for complex logic

## Deployment
- Use Vercel for deployment
- Configure proper environment variables
- Set up proper database connections
- Implement proper error monitoring
- Use proper build optimization

Remember to always prioritize user experience, security, and maintainability in your code decisions. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/antony190599) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
