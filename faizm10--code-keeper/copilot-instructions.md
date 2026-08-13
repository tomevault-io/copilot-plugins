## code-keeper

> - `/web` - Next.js frontend application

# Code Keeper - Cursor Rules

## Project Structure
- `/web` - Next.js frontend application
- `/backend` - Backend API service
- `/docs` - Documentation site

## General Guidelines

### Code Style
- Use TypeScript for all new code
- Follow ESLint and Prettier configurations
- Use meaningful variable and function names
- Write self-documenting code with clear intent
- Prefer functional programming patterns where appropriate

### TypeScript
- Always use explicit types for function parameters and return values
- Avoid `any` type - use `unknown` if type is truly unknown
- Use interfaces for object shapes, types for unions/intersections
- Enable strict mode in tsconfig.json

### React/Next.js (Web)
- Use functional components with hooks
- Prefer Server Components in Next.js App Router
- Use TypeScript for all component props
- Follow Next.js best practices for routing and data fetching
- Use Tailwind CSS for styling (already configured)
- Keep components small and focused on a single responsibility

### Backend
- Use async/await for asynchronous operations
- Implement proper error handling with try-catch blocks
- Use environment variables for configuration
- Follow RESTful API conventions
- Implement proper logging for debugging and monitoring

### Git
- Write clear, descriptive commit messages
- Use conventional commit format when possible
- Keep commits focused on a single change
- Review code before committing

### Docker
- Use multi-stage builds to reduce image size
- Don't copy unnecessary files (use .dockerignore)
- Run containers as non-root users when possible
- Use specific version tags for base images

### Testing
- Write unit tests for utility functions
- Write integration tests for API endpoints
- Write component tests for React components
- Aim for good test coverage but prioritize meaningful tests

### Documentation
- Write clear README files for each service
- Document complex algorithms and business logic
- Keep API documentation up to date
- Use JSDoc comments for public functions

### Security
- Never commit secrets or API keys
- Use environment variables for sensitive data
- Validate and sanitize user inputs
- Keep dependencies up to date
- Use HTTPS in production

### Performance
- Optimize images and assets
- Use code splitting for large applications
- Implement proper caching strategies
- Monitor and optimize database queries
- Use lazy loading where appropriate

### Error Handling
- Always handle errors gracefully
- Provide meaningful error messages to users
- Log errors with sufficient context for debugging
- Use proper HTTP status codes in API responses

### Code Review
- Review your own code before requesting review
- Be constructive in code review comments
- Address all review comments before merging
- Test changes thoroughly before submitting

## File Naming
- Use kebab-case for file names
- Use PascalCase for React components
- Use camelCase for functions and variables
- Use UPPER_SNAKE_CASE for constants

## Imports
- Group imports: external libraries, then internal modules
- Use absolute imports with path aliases when configured
- Remove unused imports

## Comments
- Write comments that explain "why", not "what"
- Keep comments up to date with code changes
- Remove commented-out code before committing

---
> Source: [faizm10/code-keeper](https://github.com/faizm10/code-keeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
