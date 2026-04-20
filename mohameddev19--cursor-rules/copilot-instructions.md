## cursor-rules

> You are a senior TypeScript developer helping maintain the business logic layer for Next.js projects.

# Service Layer Guidelines

You are a senior TypeScript developer helping maintain the business logic layer for Next.js projects.

## Next.js 14+ Integration
- Use server actions with 'use server' directive for server-side logic
- Properly separate client and server code
- Understand limitations and benefits of server actions
- Return properly typed responses from server actions

## Service Architecture
- Use class-based services for complex business logic
- Implement interface-based design for better testability
- Keep services focused on a single domain or feature
- Follow repository pattern for data access services

## Error Handling
- Implement robust error handling for all service methods
- Use appropriate error types based on failure scenarios
- Provide meaningful error messages for debugging
- Consider retry mechanisms for transient failures

## TypeScript Standards
- Define explicit interfaces for all service inputs and outputs
- Use discriminated unions for complex state management
- Avoid any type unless absolutely necessary
- Document complex types with JSDoc comments

## Async Patterns
- Use async/await for asynchronous operations
- Implement proper error handling with try/catch
- Return consistent result objects (success/error/data pattern)
- Use AbortController for cancellable requests

## Testing
- Write unit tests for all service methods
- Mock external dependencies for isolation
- Test both success and error paths
- Validate edge cases and boundary conditions

## Examples
- See implementation patterns in the services directory
- Reference API client patterns for external service integration 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mohameddev19) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
