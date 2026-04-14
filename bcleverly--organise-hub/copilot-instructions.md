## organise-hub

> - Follow Laravel's conventional directory structure

# Laravel Architecture & Project Structure

## Project Organization
- Follow Laravel's conventional directory structure
- Use [app/](mdc:app/) for application logic
- Keep [config/](mdc:config/) for configuration files
- Use [database/migrations/](mdc:database/migrations/) for database schema
- Store views in [resources/views/](mdc:resources/views/)
- Use [routes/](mdc:routes/) for route definitions

## MVC Pattern Implementation
- Models: Place in [app/Models/](mdc:app/Models/) with proper relationships
- Controllers: Use [app/Http/Controllers/](mdc:app/Http/Controllers/) with single responsibility
- Views: Organize in [resources/views/](mdc:resources/views/) with proper layouts
- Use [app/Models/User.php](mdc:app/Models/User.php) as the base authentication model

## Service Layer & Actions
- Use Laravel Actions for complex business logic
- Implement service classes for reusable business logic
- Keep controllers thin - delegate to services/actions
- Use dependency injection for better testability
- Follow SOLID principles in all classes

## Database Design
- Use migrations for version-controlled schema changes
- Implement proper foreign key constraints
- Use indexes for performance optimization
- Follow naming conventions for tables and columns
- Use soft deletes for audit trails
- Implement proper data validation at the database level

## Authentication & Authorization
- Use Laravel's built-in authentication with [app/Models/User.php](mdc:app/Models/User.php)
- Implement Gates and Policies for authorization
- Use middleware for route protection
- Implement proper role and permission systems
- Use Laravel Passkeys for modern authentication

## API Design
- Use RESTful conventions for API endpoints
- Implement proper API versioning
- Use API resources for data transformation
- Implement proper error handling and status codes
- Use API authentication (tokens, OAuth)

## Testing Strategy
- Write unit tests for models and services
- Implement feature tests for controllers
- Use [tests/](mdc:tests/) directory for all test files
- Use factories for test data generation
- Implement proper test coverage

## Configuration Management
- Use environment variables for configuration
- Keep [config/app.php](mdc:config/app.php) as the main configuration file
- Use service providers for package configuration
- Implement proper caching strategies
- Use configuration caching in production

## Frontend Integration
- Use Vite for asset compilation with [vite.config.js](mdc:vite.config.js)
- Organize CSS in [resources/css/](mdc:resources/css/)
- Organize JavaScript in [resources/js/](mdc:resources/js/)
- Use Blade templates for server-side rendering
- Implement proper asset versioning

## Performance Optimization
- Use Laravel Octane for performance
- Implement proper caching strategies
- Use database query optimization
- Implement lazy loading for relationships
- Use queue jobs for background processing

## Security Implementation
- Use CSRF protection on all forms
- Implement proper input validation
- Use prepared statements for database queries
- Implement proper session management
- Use HTTPS in production
- Follow OWASP security guidelines
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BCleverly)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BCleverly)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
