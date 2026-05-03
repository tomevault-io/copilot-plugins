## gralph

> This file contains learnings, patterns, and conventions discovered during Ralph's execution. Future iterations should reference this file to maintain consistency.

# AGENTS.md - Project Memory and Patterns

This file contains learnings, patterns, and conventions discovered during Ralph's execution. Future iterations should reference this file to maintain consistency.

## Project Structure

### File Organization
- Use `src/` for source code
- Use `tests/` for test files
- Use `docs/` for documentation
- Keep configuration files in project root

### Naming Conventions
- Use kebab-case for file names (e.g., `user-authentication.js`)
- Use camelCase for variables and functions
- Use PascalCase for classes and components
- Prefix test files with the tested file name

## Code Patterns

### Error Handling
- Use try/catch blocks for async operations
- Return descriptive error messages
- Log errors with context information
- Prefer throwing specific error types over generic ones

### Testing
- Write unit tests for all functions
- Use descriptive test names
- Test both success and failure cases
- Mock external dependencies
- Aim for >80% code coverage

### Database Operations
- Use transactions for multi-step operations
- Validate input data before database operations
- Handle connection errors gracefully
- Use prepared statements to prevent SQL injection

## Gotchas and Common Issues

### Environment Setup
- Always check for required environment variables
- Validate API keys and database connections
- Use `.env.example` for required configuration

### Performance Considerations
- Avoid N+1 queries in database operations
- Cache expensive computations
- Use pagination for large data sets
- Monitor memory usage in long-running processes

## Recent Learnings

<!-- This section gets updated automatically by Ralph iterations -->

## Dependencies and Libraries

### Core Dependencies
- List your main project dependencies here
- Include version constraints
- Note any compatibility requirements

### Development Tools
- Testing frameworks
- Linting tools
- Build tools
- Deployment scripts

## Deployment and Environment

### Staging Environment
- Configuration requirements
- Database setup
- External service integrations

### Production Environment
- Security considerations
- Monitoring setup
- Backup procedures
- Scaling requirements

## Contributing Guidelines

### Code Review Process
- Required approvals
- Automated checks
- Manual review criteria

### Git Workflow
- Branch naming conventions
- Commit message format
- Pull request templates
- Do not modify `examples` folder

---
> Source: [frizynn/gralph](https://github.com/frizynn/gralph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
