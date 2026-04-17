## associsse

> This is a Next.js 15 application with TypeScript, Prisma, and Tailwind CSS for expense and bank transaction management with multi-tenant organization support.

# Cursor Rules for Associsse Project

## Project Overview
This is a Next.js 15 application with TypeScript, Prisma, and Tailwind CSS for expense and bank transaction management with multi-tenant organization support.

## Tech Stack
- **Framework**: Next.js 15 with App Router
- **Language**: TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **Styling**: Tailwind CSS with shadcn/ui components
- **Authentication**: NextAuth.js
- **Testing**: Vitest with React Testing Library
- **File Processing**: iconv-lite for encoding detection

## Code Style & Standards

### TypeScript
- Use strict TypeScript with proper type definitions
- Prefer interfaces over types for object shapes
- Use proper generic types for reusable components
- Always define return types for functions
- Use Zod schemas for runtime validation
- Prefer `const` assertions and `as const` for literal types

### React & Next.js
- Use functional components with hooks
- Prefer `useCallback` and `useMemo` for performance optimization
- Use proper dependency arrays in useEffect
- Implement proper error boundaries
- Use Next.js App Router patterns (server components when possible)
- Prefer server actions over API routes when appropriate

### API Routes
- Always use centralized auth utilities from `@/lib/auth-utils`
- Implement proper error handling with consistent response formats
- Use Zod schemas for request validation
- Log activities using the activity system
- Follow RESTful conventions
- Use proper HTTP status codes

### Database & Prisma
- Use Prisma for all database operations
- Implement proper error handling for database operations
- Use transactions for multi-step operations
- Follow the established schema patterns
- Use proper relations and foreign keys
- Implement soft deletes when appropriate

### Authentication & Authorization
- Use `validateSession()` for basic authentication
- Use `validateMembership()` for organization access
- Use `validateRole()` for permission checks
- Always check organization membership before data access
- Implement proper multi-tenant data isolation

### File Structure
```
src/
├── app/                    # Next.js App Router pages
│   ├── [orgSlug]/         # Organization-scoped routes
│   ├── api/               # API routes
│   └── globals.css        # Global styles
├── components/            # Reusable UI components
│   └── ui/               # shadcn/ui components
├── lib/                  # Utility libraries
│   ├── __tests__/        # Library tests
│   ├── auth.ts           # NextAuth configuration
│   ├── auth-utils.ts     # Authentication utilities
│   ├── db.ts             # Prisma client
│   ├── validations.ts    # Zod schemas
│   └── activity-utils.ts # Activity logging
└── test/                 # Test configuration
```

### Component Patterns
- Use shadcn/ui components as base
- Implement proper loading states
- Use proper error handling
- Follow accessibility guidelines
- Use Tailwind CSS for styling
- Implement responsive design

### Loading States & Skeletons
- **ALWAYS use Skeleton components for loading states** instead of custom spinners or "Loading..." text
- Import Skeleton from `@/components/ui/skeleton`
- Create skeleton layouts that match the actual content structure
- Use appropriate skeleton dimensions (h-4, h-6, h-8, h-10, w-16, w-24, w-32, etc.)
- For tables: create skeleton rows with proper column widths
- For cards: use skeleton placeholders for titles, content, and actions
- For forms: use skeleton inputs and buttons
- For charts: use skeleton containers with appropriate heights
- **NEVER use**: `Loader2` spinners, custom `animate-spin` divs, or plain "Loading..." text
- **NEVER use**: `bg-gray-200 animate-pulse` custom implementations
- **ALWAYS**: Match skeleton structure to actual content layout
- **ALWAYS**: Include skeleton for header buttons and actions
- **ALWAYS**: Use consistent skeleton sizing across the application

### Testing
- Use Vitest for unit tests
- Use React Testing Library for component tests
- Mock external dependencies properly
- Test both success and error cases
- Use descriptive test names
- Group related tests with `describe` blocks

### Error Handling
- Use try-catch blocks for async operations
- Provide meaningful error messages
- Log errors appropriately
- Return proper HTTP status codes
- Handle edge cases gracefully

### Performance
- Use proper React optimization techniques
- Implement proper caching strategies
- Use database indexes appropriately
- Optimize bundle size
- Use proper image optimization

## Specific Patterns

### Bank Transactions
- Handle multiple text encodings (UTF-8, ISO 8859-2, Windows-1252)
- Use iconv-lite for encoding detection
- Generate unique hashes to prevent duplicates
- Associate negative transactions with positive expenses
- Use absolute values for amount calculations

### Multi-tenancy
- Always scope data by organization
- Use organization slug in routes
- Implement proper access controls
- Separate data by organization ID

### Activity Logging
- Log all significant user actions
- Use proper activity types
- Include relevant metadata
- Store activity records for audit trails

### File Uploads
- Validate file types and sizes
- Handle different encodings properly
- Implement proper error handling
- Use secure file processing

## Code Quality Rules

### Naming Conventions
- Use camelCase for variables and functions
- Use PascalCase for components and types
- Use kebab-case for file names
- Use descriptive names that explain intent
- Avoid abbreviations unless widely understood

### Comments & Documentation
- Write self-documenting code
- Add comments for complex business logic
- Document API endpoints
- Explain non-obvious decisions
- Keep comments up to date

### Security
- Validate all inputs
- Sanitize user data
- Use proper authentication
- Implement proper authorization
- Follow OWASP guidelines
- Use environment variables for secrets

### Performance
- Optimize database queries
- Use proper indexing
- Implement caching where appropriate
- Minimize bundle size
- Use proper image optimization
- Monitor performance metrics

## Git & Version Control
- Use conventional commit messages
- Create feature branches
- Write meaningful commit messages
- Use proper pull request descriptions
- Keep commits atomic and focused

## Environment & Configuration
- Use environment variables for configuration
- Separate development and production configs
- Use proper secret management
- Document required environment variables
- Use proper database migrations

## Dependencies
- Keep dependencies up to date
- Use exact versions for critical dependencies
- Prefer well-maintained packages
- Avoid unnecessary dependencies
- Use proper dependency management

## When Adding New Features
1. Follow the established patterns
2. Use proper TypeScript types
3. Implement proper error handling
4. Add appropriate tests
5. Update documentation
6. Consider security implications
7. Follow accessibility guidelines
8. Implement proper logging

## Code Review Checklist
- [ ] Follows established patterns
- [ ] Proper TypeScript usage
- [ ] Error handling implemented
- [ ] Tests added/updated
- [ ] Security considerations addressed
- [ ] Performance implications considered
- [ ] Documentation updated
- [ ] Accessibility guidelines followed
- [ ] **Loading states use Skeleton components (not custom spinners or "Loading..." text)**
- [ ] **Skeleton layouts match actual content structure**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wallforfry) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
