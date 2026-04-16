## sploosh-ai-hockey-analytics

> These rules provide a template for project-specific configurations that can be customized for each repository.


# Project Template Rules

These rules provide a template for project-specific configurations that can be customized for each repository.

## Project Structure

- Follow the Next.js app directory structure for the project
- Place components in the appropriate directories:
  - `/components/features` for feature-specific components
  - `/components/ui` for reusable UI components
  - `/components/layouts` for layout components
  - `/components/shared` for shared utility components
- Use `/lib` for utility functions and API clients
- Store types in `/types` or alongside the components that use them
- Place API routes in `/app/api` directory

## Code Style

- Follow TypeScript best practices and use strict type checking
- Use functional React components with hooks
- Implement proper error handling for all async operations
- Use CSS modules or Tailwind CSS for styling
- Follow the project's naming conventions:
  - PascalCase for component files and React components
  - camelCase for variables, functions, and instances
  - kebab-case for file names (except for React components)
  - UPPER_CASE for constants

## React Components

- Keep components focused on a single responsibility
- Extract reusable logic into custom hooks
- Use proper prop typing with TypeScript interfaces
- Implement error boundaries for critical components
- Optimize rendering performance with memoization where appropriate
- Follow accessibility best practices in all components

## Testing

- Write tests for new features and bug fixes
- Use Jest for unit tests and React Testing Library for component tests
- Ensure all tests pass before submitting a pull request
- Maintain test coverage at or above 80%
- Write meaningful test descriptions that explain the expected behavior

## Documentation

- Update documentation when making changes to code
- Document APIs, functions, and complex logic with JSDoc comments
- Keep README files up-to-date with current project information
- Include examples for complex components or utilities
- Document any non-obvious behavior or edge cases

## Deployment

- Follow the project's deployment process via GitHub Actions
- Test changes in development/staging environments before production
- Document any deployment-specific considerations
- Update environment variables documentation when adding new ones

## Data Handling

- Implement proper data validation for all user inputs
- Use appropriate caching strategies for API responses
- Handle loading, error, and empty states for all data fetching
- Implement proper pagination for large data sets
- Follow data privacy best practices and handle sensitive information securely

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SplooshAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
