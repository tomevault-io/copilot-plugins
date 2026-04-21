## veliano

> Development standards and practices


 # Development Standards and Best Practices

This document outlines our development standards and best practices to maintain code quality and consistency.

## Next.js 15 Specific Standards

- Use the App Router for all new route development
- Implement Server Components by default, only use Client Components when necessary
- Use Server Actions for form submissions and data mutations
- Follow proper async/await patterns for Web APIs (cookies, headers)
- Use middleware for authentication and protection patterns

## TypeScript Standards

- Use TypeScript for all new code
- Define explicit types for function parameters and return values
- Use interfaces for complex objects and type aliases for simple types
- Avoid `any` type wherever possible
- Use proper type imports and exports

## Authentication Testing Standards

- Use mock Supabase client for authentication tests
- Implement proper test doubles for Supabase responses
- Test all authentication flows:
  - Sign up
  - Sign in
  - Password reset
  - Session management
  - Error cases
- Use proper TypeScript types for test data
- Follow AAA pattern (Arrange, Act, Assert)
- Implement proper cleanup in test teardown
- Use environment variables for test configuration

## Supabase Integration

- Use the repository pattern for database access
- Follow Row Level Security (RLS) policies for data protection
- Create proper TypeScript types for database schema
- Use Server Components for database queries
- Implement proper error handling for database operations

## Code Organization

- Use functional components instead of class components
- Follow the single responsibility principle
- Keep components small and focused
- Use custom hooks for shared logic
- Separate business logic from UI components

## Performance Optimization

- Implement proper loading states for async operations
- Use image optimization for all images
- Implement code splitting with dynamic imports
- Minimize client-side JavaScript
- Use proper caching strategies for static content

## Deployment

- Use Vercel for deployment
- Set up proper environment variables
- Implement pre-deployment checks
- Follow the deployment checklist in `.cursor/DEPLOY.md`
- Monitor performance after deployment

# Development Standards

## Current Focus Areas

### 1. Performance Monitoring
- Implement comprehensive monitoring for all critical paths
- Track Core Web Vitals (LCP, FID, CLS)
- Monitor API and database performance
- Ensure proper error tracking and reporting
- Maintain type safety in monitoring implementations

### 2. Type Safety
- Zero tolerance for `any` types
- Explicit type definitions for all functions
- Proper error type handling
- Type-safe API responses
- Validated form inputs with type checking

### 3. Error Handling
- Implement error boundaries at appropriate levels
- Track errors with proper context
- Handle both client and server errors gracefully
- Provide user-friendly error messages
- Log errors with appropriate severity levels

## Code Quality Requirements

### TypeScript Standards
- Strict mode enabled
- No implicit any
- No unsafe type assertions
- Proper generic type usage
- Interface-first approach
- Proper error type definitions

### Component Standards
- Functional components only
- Props interface definitions
- Error boundary implementation
- Loading state handling
- Proper type checking

### Testing Requirements
- Unit tests for utilities
- Integration tests for flows
- Error case coverage
- Performance monitoring tests
- Type checking in tests

## Performance Standards

### Monitoring Requirements
- Core Web Vitals tracking
- API performance monitoring
- Database query monitoring
- Error tracking and reporting
- Performance regression testing

### Optimization Requirements
- Code splitting
- Image optimization
- Font optimization
- API response caching
- Database query optimization

## Documentation Requirements

### Code Documentation
- Interface documentation
- Function documentation
- Error handling documentation
- Performance considerations
- Type definitions

### API Documentation
- OpenAPI/Swagger specs
- Error responses
- Type definitions
- Performance characteristics
- Security considerations

## Current Phase Standards (Phase 2 → 3 Transition)

### Code Quality Requirements

1. TypeScript
   - Strict mode enabled
   - No `any` types in core functionality
   - Proper type exports for all components
   - Type-safe database operations
   - Comprehensive interface definitions

2. Component Structure
   - Server Components by default
   - Client components only when necessary
   - Proper error boundaries
   - Loading state handling
   - Optimistic updates where appropriate

3. Testing Requirements
   - Unit tests for all components
   - Integration tests for user flows
   - RLS policy tests
   - Error handling tests
   - Performance tests

### Database Standards

1. Supabase Integration
   - Type-safe queries using generated types
   - RLS policies for all tables
   - Proper error handling
   - Optimistic updates with fallbacks
   - Transaction support where needed

2. Schema Management
   - Migrations for all changes
   - Proper indexing
   - Foreign key constraints
   - Cascade behaviors defined
   - Backup strategies

### Error Handling

1. Client-Side
   - Use Error Boundaries for component-level errors
   - Implement toast notifications for user feedback
   - Use loading states during async operations
   - Implement retry mechanisms for failed operations
   - Support offline state gracefully

2. Server-Side
   - Use structured error responses
   - Implement proper HTTP status codes
   - Use type-safe error tracking with Sentry
   - Implement rate limiting
   - Handle validation errors consistently

3. Error Tracking Implementation
   ```typescript
   // Example of type-safe error tracking
   import { trackError, trackMessage } from '@/lib/utils/error-tracking';

   try {
     // Operation that might fail
   } catch (error) {
     trackError(error as Error, {
       severity: 'error',
       context: {
         component: 'AddressManager',
         action: 'createAddress',
         userId: currentUser.id,
         additionalData: { addressData }
       }
     });
   }
   ```

4. Error Boundaries
   ```typescript
   // Example of component error boundary
   const ComponentErrorBoundary: React.FC<{ children: React.ReactNode }> = ({ children }) => {
     return (
       <ErrorBoundary
         fallback={({ error }) => {
           trackError(error, {
             severity: 'error',
             context: { component: 'ComponentName' }
           });
           return <ErrorFallback />;
         }}
       >
         {children}
       </ErrorBoundary>
     );
   };
   ```

5. Error Recovery Patterns
   - Implement optimistic updates with rollback
   - Use retry queues for failed operations
   - Cache valid data for offline support
   - Provide clear user feedback
   - Log recovery attempts

### Performance Standards

1. Core Web Vitals
   - LCP < 2.5s
   - FID < 100ms
   - CLS < 0.1

2. API Performance
   - Response time < 200ms
   - Caching strategy
   - Query optimization
   - Connection pooling
   - Rate limiting

### Security Standards

1. Authentication
   - Protected routes
   - Session management
   - Rate limiting
   - CSRF protection
   - XSS prevention

2. Data Access
   - RLS policies
   - Input validation
   - Output sanitization
   - Audit logging
   - Access controls

### Documentation Requirements

1. Code Documentation
   - Component documentation
   - Type definitions
   - API endpoints
   - Database schema
   - Security policies

2. Testing Documentation
   - Test coverage reports
   - Integration test scenarios
   - Performance test results
   - Security audit results

## Phase 3 Preparation Standards

### Product Management

1. Database Schema
   - Product table design
   - Category relationships
   - Inventory tracking
   - Image storage
   - Search indexing

2. Component Architecture
   - Product list components
   - Product detail views
   - Category navigation
   - Search interface
   - Admin controls

### Testing Strategy

1. Unit Tests
   - Component testing
   - Utility function testing
   - Hook testing
   - State management testing

2. Integration Tests
   - User flows
   - Admin flows
   - Search functionality
   - Filter operations
   - Error scenarios

### Performance Optimization

1. Image Optimization
   - Lazy loading
   - Proper sizing
   - Format optimization
   - CDN usage
   - Caching strategy

2. Search Performance
   - Index optimization
   - Query caching
   - Result pagination
   - Filter optimization
   - Sort performance

## Commit Guidelines

1. Commit Message Format
   ```
   <type>(<scope>): <description>

   [optional body]

   [optional footer]
   ```

2. Types
   - feat: New feature
   - fix: Bug fix
   - refactor: Code change
   - style: Style updates
   - test: Test updates
   - docs: Documentation
   - chore: Maintenance

3. Scope Examples
   - auth
   - profile
   - api
   - db
   - ui
   - test

4. Description Guidelines
   - Present tense
   - Imperative mood
   - No period at end
   - Max 72 characters

## Code Review Standards

1. Review Checklist
   - Type safety
   - Error handling
   - Performance impact
   - Security implications
   - Test coverage
   - Documentation

2. Review Priority
   - Security issues
   - Type errors
   - Performance issues
   - Code style
   - Documentation

## Deployment Standards

1. Pre-deployment Checks
   - All tests passing
   - Type checking
   - Lint checking
   - Build success
   - Performance audit

2. Deployment Process
   - Staging deployment
   - Smoke tests
   - Performance verification
   - Security scan
   - Production deployment

## Testing Standards

### Supabase Testing
1. Use Type-Safe Mocks
   - Import test utilities from `tests/utils/supabase-test-utils`
   - Use proper generic types for table data
   - Implement mock builders for common scenarios
   - Handle error cases explicitly

2. Test Data Structure
   ```typescript
   type MockData<T> = {
     data: T | null;
     error: Error | null;
     status: number;
     statusText: string;
     count: number | null;
   };
   ```

3. Common Test Scenarios
   - Basic CRUD operations
   - Error handling
   - Network failures
   - Optimistic updates
   - Loading states

4. Mock Implementation
   ```typescript
   vi.mock('@/lib/supabase-client', () => ({
     supabase: createMockSupabase('tableName', mockData)
   }));
   ```

### Integration Testing Best Practices
1. Component Setup
   - Clear mocks before each test
   - Set up default mock responses
   - Use helper functions for common operations

2. Error Handling
   - Test network errors
   - Test validation errors
   - Test optimistic update failures
   - Verify error messages
   - Check recovery flows

3. UI Verification
   - Use proper aria roles
   - Test loading states
   - Verify success messages
   - Check error displays
   - Test user interactions

4. Test Organization
   - Group related tests
   - Use descriptive test names
   - Follow AAA pattern
   - Include cleanup
   - Document complex scenarios

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ai-Eli-ML) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
