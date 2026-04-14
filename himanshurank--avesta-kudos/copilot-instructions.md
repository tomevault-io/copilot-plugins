## avesta-kudos

> This rule defines comprehensive coding standards and best practices for the project across all architectural layers. It covers code style, documentation, error handling, performance, and security guidelines. Includes specific practices for domain, application, infrastructure, and presentation layers. Also provides testing best practices for unit, integration, and E2E tests. Reference this rule when implementing new features or refactoring existing code to maintain consistency and quality.

# Project Best Practices

## Common Best Practices

### Code Style & Documentation

- Use TypeScript strict mode
- Document public interfaces and complex functions with JSDoc
- Keep functions small and focused (max 20-30 lines)
- Use meaningful variable/function names (no abbreviations)
- Document complex business logic with inline comments
- Use prettier for consistent formatting
- Follow conventional commits (feat:, fix:, chore:, etc.)

### Error Handling

- Use custom error classes for domain-specific errors
- Always include error context in error messages
- Log errors with appropriate severity levels
- Handle async errors with try/catch
- Provide user-friendly error messages in UI

### Performance

- Memoize expensive computations
- Use React.memo for pure components
- Implement proper cleanup in useEffect
- Lazy load routes and heavy components
- Use proper keys in lists
- Debounce frequent events

### Security

- Never store sensitive data in localStorage
- Sanitize user inputs
- Use environment variables for secrets
- Implement proper CORS policies
- Use HTTPS for all API calls
- Validate all API responses

## Layer-Specific Best Practices

### Domain Layer

- Keep entities immutable
- Use value objects for complex attributes
- Implement validation in constructors
- Use factory methods for complex object creation
- Keep business rules in domain objects
- Document invariants and constraints

### Application Layer

- Use interfaces for dependencies
- Implement command/query separation
- Keep services stateless
- Use DTOs for data transfer
- Document service methods
- Handle all possible error cases

### Infrastructure Layer

- Implement retry logic for external calls
- Cache expensive operations
- Use dependency injection
- Implement proper logging
- Handle network errors gracefully
- Document API endpoints

### Presentation Layer

- Use atomic design principles
- Keep components pure when possible
- Implement proper loading states
- Handle error boundaries
- Use proper prop types
- Document component props

## Testing Best Practices

### Unit Tests

- Test one thing per test
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)
- Mock external dependencies
- Test edge cases and error scenarios
- Keep tests independent

### Integration Tests

- Test complete features
- Use test databases
- Clean up after tests
- Mock external services
- Test happy and error paths
- Document test scenarios

### E2E Tests

- Test critical user flows
- Use realistic test data
- Test on multiple browsers
- Include visual regression
- Document test scenarios
- Use page objects

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Himanshurank)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Himanshurank)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
