## team-workspace-laravel

> 1. **Request Validation**


# Project Rules and Guidelines

## Code Quality & Standards

### PHP/Laravel Backend Rules

1. **Request Validation**
   - Always use Form Request classes for validation
   - Follow the existing pattern: extend `BaseRequest` or `FormRequest`
   - Include proper validation rules with appropriate messages
   - Handle unique constraints with `Rule::unique()->ignore($id)` for updates

2. **Database Operations**
   - Always use database transactions for complex operations
   - Use `DB::beginTransaction()` and `DB::commit()` pattern
   - Include proper rollback handling in catch blocks
   - Follow naming conventions for migrations and models

3. **API Responses**
   - Use consistent response format with `success()` and `error()` methods
   - Include proper HTTP status codes
   - Handle both API and web requests appropriately
   - Always return meaningful error messages

4. **Security**
   - Validate all input data
   - Use Laravel's built-in security features
   - Implement proper authorization checks
   - Never expose sensitive data in responses

### Frontend (Next.js/React) Rules

1. **Component Structure**
   - Use TypeScript for all components
   - Follow the existing UI component patterns
   - Use Zod for form validation schemas
   - Implement proper error handling

2. **Form Validation**
   - Use `react-hook-form` with `zodResolver`
   - Create validation schemas with meaningful error messages
   - Handle both client and server validation
   - Provide user-friendly error feedback

3. **API Integration**
   - Use proper TypeScript types for API responses
   - Handle loading states appropriately
   - Implement proper error handling
   - Use consistent API calling patterns

## Development Guidelines

### File Organization

1. **Backend Structure**
   ```
   app/Http/Requests/     # Form validation classes
   app/Models/           # Eloquent models
   app/Http/Controllers/ # API controllers
   database/migrations/  # Database schema
   ```

2. **Frontend Structure**
   ```
   src/components/       # Reusable UI components
   src/types/           # TypeScript type definitions
   src/lib/             # Utility functions
   ```

### Naming Conventions

1. **PHP Classes**
   - Models: `PascalCase` (e.g., `PayrollRun`)
   - Controllers: `PascalCase` + `Controller` (e.g., `PayrollController`)
   - Requests: `PascalCase` + `Request` (e.g., `PayrollRunRequest`)

2. **Database**
   - Tables: `snake_case` plural (e.g., `payroll_runs`)
   - Columns: `snake_case` (e.g., `created_at`)
   - Foreign keys: `table_id` (e.g., `driver_id`)

3. **Frontend**
   - Components: `PascalCase` (e.g., `AddVehicleModal`)
   - Files: `kebab-case` (e.g., `add-vehicle-modal.tsx`)
   - Variables: `camelCase` (e.g., `vehicleData`)

## Validation Rules

### Common Validation Patterns

1. **Required Fields**
   ```php
   'name' => ['required', 'string', 'max:255', 'min:3']
   ```

2. **Email Validation**
   ```php
   'email' => ['required', 'email', 'unique:table,email']
   ```

3. **Unique with Update**
   ```php
   'email' => ['required', 'email', Rule::unique('table', 'email')->ignore($id)]
   ```

4. **Numeric Values**
   ```php
   'price' => ['required', 'numeric', 'min:0']
   ```

### Frontend Validation with Zod

```typescript
const schema = z.object({
  name: z.string().min(1, { message: 'الاسم مطلوب' }),
  email: z.string().email({ message: 'البريد الإلكتروني غير صالح' }),
  phone: z.string().min(10, { message: 'رقم الهاتف غير صالح' })
});
```

## Error Handling

### Backend Error Handling

```php
try {
    DB::beginTransaction();
    // Operations
    DB::commit();
    return $this->success($data, 'Success message');
} catch (\Exception $e) {
    DB::rollback();
    return $this->error([], $e->getMessage());
}
```

### Frontend Error Handling

```typescript
try {
    const response = await api.post('/endpoint', data);
    // Handle success
} catch (error) {
    console.error('Error:', error);
    // Show user-friendly error message
}
```

## Testing Guidelines

1. **Backend Testing**
   - Write unit tests for models and services
   - Test API endpoints with different scenarios
   - Test validation rules comprehensively
   - Mock external dependencies

2. **Frontend Testing**
   - Test component rendering
   - Test form validation
   - Test user interactions
   - Test API integration

## Performance Guidelines

1. **Database Optimization**
   - Use appropriate indexes
   - Optimize queries with eager loading
   - Use pagination for large datasets
   - Cache frequently accessed data

2. **Frontend Optimization**
   - Use React.memo for expensive components
   - Implement proper loading states
   - Optimize bundle size
   - Use lazy loading where appropriate

## Documentation Requirements

1. **Code Comments**
   - Document complex business logic
   - Explain non-obvious code decisions
   - Use PHPDoc for PHP methods
   - Use JSDoc for TypeScript functions

2. **API Documentation**
   - Document all API endpoints
   - Include request/response examples
   - Document error codes and messages
   - Keep documentation up to date

## Deployment Guidelines

1. **Environment Configuration**
   - Use environment variables for sensitive data
   - Maintain separate configs for dev/staging/production
   - Document required environment variables

2. **Build Process**
   - Run tests before deployment
   - Optimize assets for production
   - Use proper caching strategies
   - Monitor application performance

## Frontend Development Rules

### Next.js Specific Guidelines

1. **App Router Structure**
   - Use the app directory structure for new pages
   - Implement proper loading and error components
   - Use server components by default, client components when needed
   - Follow the page.tsx, layout.tsx, loading.tsx, error.tsx pattern

2. **State Management**
   - Use React hooks for local state management
   - Implement Context API for shared state when needed
   - Use react-query/tanstack-query for server state management
   - Avoid prop drilling with proper state lifting

3. **Styling Guidelines**
   - Use Tailwind CSS for styling
   - Follow mobile-first responsive design principles
   - Use CSS modules for component-specific styles when needed
   - Maintain consistent spacing and color schemes

4. **Performance Optimization**
   - Use Next.js Image component for images
   - Implement proper code splitting with dynamic imports
   - Use React.memo for expensive components
   - Optimize bundle size with proper imports

### UI/UX Guidelines

1. **Component Library**
   - Use shadcn/ui components as the base
   - Maintain consistent design tokens
   - Create reusable composite components
   - Follow accessibility best practices (ARIA labels, keyboard navigation)

2. **Form Design**
   - Use react-hook-form for form management
   - Implement proper validation feedback
   - Show loading states during form submission
   - Provide clear success/error messages

3. **Responsive Design**
   - Mobile-first approach
   - Test on multiple screen sizes
   - Use appropriate breakpoints
   - Ensure touch-friendly interfaces

## Database Design Rules

### Schema Design

1. **Table Structure**
   - Use singular nouns for table names (user, not users)
   - Include created_at and updated_at timestamps
   - Use soft deletes where appropriate with deleted_at
   - Implement proper foreign key constraints

2. **Indexing Strategy**
   - Index frequently queried columns
   - Create composite indexes for multi-column queries
   - Index foreign key columns
   - Monitor and optimize query performance

3. **Data Integrity**
   - Use appropriate data types
   - Set proper column constraints (NOT NULL, UNIQUE)
   - Implement cascading deletes where appropriate
   - Use database-level defaults for common fields

### Migration Best Practices

1. **Migration Structure**
   - Use descriptive migration names
   - Include rollback methods (down())
   - Test migrations on development data
   - Use schema builder methods, not raw SQL

2. **Data Migrations**
   - Separate schema and data migrations
   - Handle large datasets with chunking
   - Backup data before destructive changes
   - Test rollback procedures

## API Design Standards

### RESTful API Guidelines

1. **URL Structure**
   - Use nouns for resources, not verbs
   - Follow REST conventions (GET, POST, PUT, DELETE)
   - Use proper HTTP status codes
   - Implement consistent API versioning

2. **Request/Response Format**
   - Use JSON for data exchange
   - Implement consistent response structure
   - Include proper error codes and messages
   - Use pagination for list endpoints

3. **Authentication & Authorization**
   - Use Laravel Sanctum for API authentication
   - Implement role-based access control
   - Validate permissions at controller level
   - Log authentication attempts

### Error Handling

1. **HTTP Status Codes**
   - 200: Success
   - 201: Created
   - 400: Bad Request
   - 401: Unauthorized
   - 403: Forbidden
   - 404: Not Found
   - 422: Validation Error
   - 500: Server Error

2. **Error Response Format**
   ```json
   {
     "success": false,
     "message": "Error description",
     "errors": {
       "field": ["Validation error message"]
     },
     "code": "ERROR_CODE"
   }
   ```

## Security Guidelines

### Backend Security

1. **Input Validation**
   - Validate all user inputs
   - Sanitize data before database operations
   - Use parameterized queries
   - Implement CSRF protection

2. **Authentication Security**
   - Use strong password policies
   - Implement rate limiting for login attempts
   - Use secure session management
   - Implement two-factor authentication where needed

3. **Data Protection**
   - Encrypt sensitive data at rest
   - Use HTTPS for all communications
   - Implement proper access controls
   - Regular security audits

### Frontend Security

1. **XSS Prevention**
   - Sanitize user inputs
   - Use Content Security Policy
   - Avoid dangerouslySetInnerHTML
   - Validate data from APIs

2. **Authentication Handling**
   - Store tokens securely
   - Implement automatic token refresh
   - Handle expired sessions gracefully
   - Logout on security events

## Development Workflow

### Git Workflow

1. **Branch Naming**
   - feature/feature-name
   - bugfix/bug-description
   - hotfix/critical-fix
   - release/version-number

2. **Commit Messages**
   - Use conventional commit format
   - Include ticket/issue numbers
   - Write descriptive commit messages
   - Keep commits atomic and focused

3. **Code Review Process**
   - All code must be reviewed before merge
   - Run tests before requesting review
   - Address all review comments
   - Update documentation if needed

### Environment Management

1. **Environment Variables**
   - Use .env files for configuration
   - Never commit sensitive data
   - Document required environment variables
   - Use different configs for different environments

2. **Dependency Management**
   - Lock dependency versions
   - Regular security updates
   - Remove unused dependencies
   - Document version requirements

## Monitoring & Logging

### Application Monitoring

1. **Performance Monitoring**
   - Monitor API response times
   - Track database query performance
   - Monitor memory and CPU usage
   - Set up alerts for critical metrics

2. **Error Tracking**
   - Log all application errors
   - Use structured logging format
   - Include context in error logs
   - Set up error notifications

3. **User Analytics**
   - Track user interactions
   - Monitor feature usage
   - Analyze user flows
   - Respect user privacy

### Logging Standards

1. **Log Levels**
   - DEBUG: Detailed debug information
   - INFO: General information
   - WARNING: Warning messages
   - ERROR: Error conditions
   - CRITICAL: Critical conditions

2. **Log Format**
   ```json
   {
     "timestamp": "2024-01-01T12:00:00Z",
     "level": "INFO",
     "message": "User logged in",
     "context": {
       "user_id": 123,
       "ip_address": "192.168.1.1"
     }
   }
   ```

## Deployment Guidelines

### Production Deployment

1. **Build Process**
   - Run all tests before deployment
   - Build optimized production assets
   - Verify environment configuration
   - Create deployment backups

2. **Database Migrations**
   - Test migrations on staging first
   - Backup database before migrations
   - Run migrations during low-traffic periods
   - Have rollback plan ready

3. **Monitoring Post-Deployment**
   - Monitor application logs
   - Check performance metrics
   - Verify functionality works
   - Monitor error rates

### Staging Environment

1. **Environment Setup**
   - Mirror production configuration
   - Use production-like data
   - Test deployment process
   - Validate integrations

2. **Testing Process**
   - Full regression testing
   - Performance testing
   - Security testing
   - User acceptance testing

## Code Quality Standards

### Code Review Checklist

1. **Functionality**
   - Code works as intended
   - Edge cases are handled
   - Error conditions are managed
   - Performance is acceptable

2. **Code Quality**
   - Code is readable and maintainable
   - Follows naming conventions
   - Proper error handling
   - Adequate test coverage

3. **Security**
   - Input validation implemented
   - No security vulnerabilities
   - Proper authorization checks
   - Sensitive data protected

### Documentation Requirements

1. **Code Documentation**
   - Document complex business logic
   - Include API documentation
   - Update README files
   - Document configuration changes

2. **User Documentation**
   - Update user guides
   - Document new features
   - Include troubleshooting guides
   - Maintain changelog

---

*These comprehensive rules should be followed consistently across the entire project to ensure code quality, security, and maintainability.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muhammed2020osman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/muhammed2020osman)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
