## ton-scaffolding

> Brief description of your project, its purpose, and main technologies used.

# Cursor Rules for This Project

## Project Overview
Brief description of your project, its purpose, and main technologies used.

## Tech Stack
- **Frontend**: [e.g., React, Vue, Angular]
- **Backend**: [e.g., Node.js, Python, Go]
- **Database**: [e.g., PostgreSQL, MongoDB]
- **Testing**: [e.g., Jest, Pytest]
- **Build Tools**: [e.g., Webpack, Vite]

## Code Style Guidelines

### General Principles
- Write clean, readable, and maintainable code
- Follow existing patterns and conventions in the codebase
- Prefer composition over inheritance
- Keep functions small and focused on a single responsibility
- Use descriptive variable and function names

### Language-Specific Conventions

#### JavaScript/TypeScript
- Use ES6+ features appropriately
- Prefer const over let, avoid var
- Use async/await over callbacks
- Always use strict equality (===)
- Destructure objects and arrays when appropriate

#### Naming Conventions
- **Variables**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Classes**: PascalCase
- **Files**: kebab-case or camelCase (be consistent)
- **React Components**: PascalCase

### File Structure
```
src/
  ├── components/     # Reusable UI components
  ├── pages/          # Page components
  ├── utils/          # Utility functions
  ├── services/       # API and external service integrations
  ├── types/          # TypeScript type definitions
  └── styles/         # Global styles
```

## Development Workflow

### Before Making Changes
1. Understand the existing code structure
2. Check for existing patterns and utilities
3. Review related tests
4. Consider performance implications

### When Writing Code
1. Write tests for new functionality
2. Update documentation if needed
3. Ensure code passes linting
4. Handle errors appropriately
5. Add logging where appropriate

### After Making Changes
1. Run tests: `npm test`
2. Run linter: `npm run lint`
3. Run type checking: `npm run typecheck`
4. Verify the build: `npm run build`

## Testing Guidelines
- Write unit tests for utility functions
- Write integration tests for API endpoints
- Aim for meaningful test coverage, not 100%
- Use descriptive test names that explain what is being tested
- Follow the Arrange-Act-Assert pattern

## Git Commit Conventions
- Use present tense ("Add feature" not "Added feature")
- Use imperative mood ("Move cursor to..." not "Moves cursor to...")
- Limit first line to 72 characters
- Reference issues and pull requests when applicable

### Commit Types
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes (formatting, missing semicolons, etc.)
- `refactor:` Code refactoring
- `test:` Adding or updating tests
- `chore:` Maintenance tasks

## Security Best Practices
- Never hardcode sensitive information
- Always validate user input
- Use environment variables for configuration
- Sanitize data before rendering
- Keep dependencies up to date
- Follow OWASP guidelines

## Performance Considerations
- Optimize images and assets
- Implement lazy loading where appropriate
- Minimize bundle size
- Use caching strategies
- Profile and monitor performance

## Error Handling
- Always handle promises and async operations
- Provide meaningful error messages
- Log errors appropriately
- Implement fallback UI for error states
- Never expose sensitive information in error messages

## Documentation Standards
- Document complex logic with inline comments
- Maintain up-to-date README
- Document API endpoints clearly
- Include examples in documentation
- Keep documentation close to code

## Dependencies
- Prefer well-maintained packages
- Check package size before adding
- Review security advisories
- Document why a dependency was added
- Keep dependencies up to date

## Accessibility (A11y)
- Use semantic HTML
- Provide alt text for images
- Ensure keyboard navigation
- Maintain proper heading hierarchy
- Test with screen readers

## DO's
- ✅ Follow existing patterns
- ✅ Write self-documenting code
- ✅ Consider edge cases
- ✅ Optimize for readability
- ✅ Ask for clarification when needed

## DON'Ts
- ❌ Don't duplicate code unnecessarily
- ❌ Don't ignore error handling
- ❌ Don't commit sensitive data
- ❌ Don't skip tests
- ❌ Don't over-engineer solutions

## Project-Specific Rules
[Add any project-specific rules, patterns, or considerations here]

## Common Commands
```bash
# Development
npm run dev           # Start development server
npm run build         # Build for production
npm run test          # Run tests
npm run lint          # Run linter
npm run typecheck     # Run type checking

# Git
git status           # Check current status
git add .            # Stage all changes
git commit -m ""     # Commit changes
git push             # Push to remote
```

## Resources
- [Project Documentation](link-to-docs)
- [API Documentation](link-to-api-docs)
- [Design System](link-to-design-system)
- [Contributing Guide](link-to-contributing)

## Notes for AI Assistants
When working in this codebase:
1. Always check existing implementations before creating new ones
2. Maintain consistency with established patterns
3. Consider the impact of changes on the entire system
4. Prioritize code quality over quick fixes
5. Ask for clarification if requirements are unclear

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlphaTONCapital) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
