## safespace

> ├── app/                    # Main application code


# SafeSpace Global Code Preferences & Structure

## 1. Project Structure

### Root Level

```
safespace/
├── app/                    # Main application code
├── prisma/                 # Database schema and migrations
├── public/                # Static assets
└── scripts/               # Utility and build scripts
```

### App Directory Structure

```
app/
├── components/            # Reusable UI components
│   ├── __tests__/         # Component tests
│   ├── app-sidebar/       # Navigation sidebar
│   ├── post/              # Post-related components
│   └── ui/                # Base UI components (buttons, inputs, etc.)
├── dashboard/             # Dashboard routes and components
├── db/                    # Database access layer
│   └── repositories/      # Data access patterns
├── generated/             # Auto-generated code
│   └── prisma/            # Prisma client types
├── hooks/                 # Custom React hooks
│   └── __tests__/         # Tests for hooks
├── layouts/               # Layout components
├── lib/                   # Shared libraries
│   ├── api/               # API client configuration
│   ├── error/             # Error handling utilities
│   └── schemas/           # Validation schemas
├── routes/                # Application routes
│   ├── +types/            # Route-specific types
│   ├── api/               # API routes
│   ├── auth/              # Authentication routes
│   └── dashboard/         # Dashboard routes
├── services/              # Business logic
│   └── api.client/        # API client implementation
├── stores/                # State management
├── test/                  # Test utilities
├── types/                 # Global TypeScript types
└── utils/                 # Utility functions
```

## 2. Code Style & Conventions

### TypeScript

- Use TypeScript with strict mode enabled
- Prefer interfaces for public API definitions
- Use type aliases for complex types
- Enable strict null checks
- Use `readonly` for immutable properties
- Use `type` for utility types and `interface` for object shapes

### Naming Conventions

- **Files**: kebab-case for file names
- **Components**: PascalCase (e.g., `UserProfile.tsx`)
- **Hooks**: `use` prefix (e.g., `useAuth.ts`)
- **Types/Interfaces**: PascalCase with `I` prefix (e.g., `IUser`)
- **Variables/Functions**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Boolean variables**: prefix with `is`, `has`, `should` (e.g., `isLoading`)

### React Components

- Use functional components with TypeScript
- Prefer named exports over default exports
- Keep components small and focused
- Use React.memo for performance optimization when needed
- Destructure props at the top of the component

### Styling

- Use Tailwind CSS for styling
- Prefer utility classes over custom CSS
- Use `@apply` for repeated utility patterns
- Keep component-specific styles in the same directory as the component

## 3. State Management

### Local State

- Use `useState` for simple component state
- Use `useReducer` for complex state logic

### Global State

- Use React Context for global state that doesn't change often
- Consider Zustand for complex global state
- Keep state as close to where it's used as possible

## 4. API Layer

### API Client

- Use a centralized API client
- Implement request/response interceptors
- Handle errors consistently
- Include request cancellation

### Data Fetching

- Create custom hooks for API calls
- Implement proper loading and error states
- Use optimistic updates where appropriate

## 5. Error Handling

### Client-Side

- Use error boundaries for React component errors
- Implement proper error messages for users
- Use utilities created for consistency

### Server-Side

- Use HTTP status codes appropriately
- Return consistent error responses
- Log detailed errors server-side
- Sanitize error messages for production

## 6. Testing

### Unit Tests

- Use Vitest for unit testing
- Test business logic in isolation
- Aim for high test coverage of core functionality

### Integration Tests

- Test component interactions
- Mock API responses
- Test error states

### E2E Tests

- Use Playwright for end-to-end tests
- Test critical user flows

## 7. Security

### Authentication

- Use secure session management
- Implement proper CSRF protection
- Use HTTP-only cookies for tokens
- Implement rate limiting

### Data Protection

- Sanitize all user inputs
- Use parameterized queries for database access
- Implement proper CORS policies
- Encrypt sensitive data

## 8. Performance

### Code Splitting

- Use dynamic imports for large components
- Implement route-based code splitting
- Lazy load non-critical components

### Optimization

- Use React.memo for expensive renders
- Implement virtualization for large lists
- Optimize images and assets
- Use proper caching headers

## 9. Documentation

### Code Documentation

- Use JSDoc for public APIs
- Document complex business logic
- Keep README files up to date

### API Documentation

- Use OpenAPI/Swagger for API documentation
- Document request/response schemas
- Include example requests

## 10. Development Workflow

### Git

- Use feature branches
- Write meaningful commit messages
- Use pull requests for code review
- Squash and merge feature branches

### Code Quality

- Use ESLint for code linting
- Use Prettier for code formatting
- Run type checking in CI
- Enforce code style with pre-commit hooks

## 11. Environment Configuration

### Environment Variables

- Use `.env` files for local development
- Document all required environment variables
- Never commit sensitive data to version control

### Build & Deploy

- Use a CI/CD pipeline
- Automate testing and deployment
- Implement feature flags for gradual rollouts

## 12. Monitoring & Observability

### Logging

- Use structured logging
- Include request IDs for tracing
- Log important business events

### Monitoring

- Set up error tracking
- Monitor performance metrics
- Set up alerts for critical issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hillsrion) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
