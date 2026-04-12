## rails-instructions

> Rails Service Objects Guide


# Rails Service Objects Guide

## Core Philosophy
- Service objects encapsulate complex business operations
- Keep services focused on single responsibilities
- Make services easy to test and understand
- Use services to keep models and controllers thin
- Design for reusability and composability

## When to Use Service Objects
- Complex operations spanning multiple models
- External API integrations
- Multi-step processes with transactions
- Business logic that doesn't fit naturally in models
- Operations requiring detailed error handling
- Background job processing logic

## Service Design Principles
- One service, one responsibility
- Clear, descriptive naming that explains the operation
- Consistent interface across services
- Explicit dependencies and parameters
- Predictable return values and error handling

## Naming Conventions
- Use verb-based names that describe the action
- Include the domain context in the name
- Examples: `UserRegistrationService`, `PaymentProcessor`, `OrderFulfillment`
- Keep names specific and intention-revealing
- Avoid generic names like `UserService`

## Service Structure
- Include ActiveModel modules for validations when needed
- Define clear public interface
- Keep implementation details private
- Use dependency injection for testability
- Document complex logic thoroughly

## Input Handling
- Validate inputs early and clearly
- Use ActiveModel::Attributes for parameter handling
- Define explicit attribute types
- Provide meaningful validation messages
- Handle optional parameters gracefully

## Return Values
- Return consistent, predictable results
- Use success/failure pattern for operations
- Include relevant data in responses
- Provide clear error information
- Consider using Result objects for complex returns

## Error Handling
- Anticipate and handle expected failures
- Use exceptions for unexpected errors
- Provide actionable error messages
- Log errors with appropriate context
- Design for graceful degradation

## Transaction Management
- Wrap related database changes in transactions
- Handle rollbacks appropriately
- Keep transactions as short as possible
- Document transaction boundaries
- Test rollback scenarios

## Testing Service Objects
- Test services in isolation
- Mock external dependencies
- Verify success and failure paths
- Test edge cases thoroughly
- Ensure idempotency where appropriate

## Dependency Injection
- Inject dependencies through initialization
- Avoid hard-coded dependencies
- Use interfaces for external services
- Keep services loosely coupled
- Document required dependencies

## Service Composition
- Build complex operations from simple services
- Keep services composable and reusable
- Avoid service objects calling other services directly
- Use orchestrator services for complex workflows
- Maintain clear boundaries between services

## Background Job Integration
- Design services to work synchronously and asynchronously
- Keep job classes thin, delegate to services
- Handle job failures gracefully
- Make services idempotent for retries
- Log job execution appropriately

## External API Integration
- Isolate API logic in dedicated services
- Handle API failures gracefully
- Implement appropriate retry logic
- Cache API responses when sensible
- Monitor API usage and errors

## Performance Considerations
- Profile service execution time
- Optimize database queries
- Consider caching expensive operations
- Batch operations when possible
- Monitor resource usage

## Service Organization
- Group related services in modules
- Keep service files in app/services
- Organize by domain or feature
- Maintain consistent file structure
- Document service relationships

## Common Patterns
- Form objects for complex input handling
- Query objects for complex database queries
- Presenter objects for view logic
- Policy objects for authorization rules
- Workflow objects for multi-step processes

## Anti-Patterns to Avoid
- God services that do too much
- Services that mirror controller actions
- Anemic services with no real logic
- Services tightly coupled to specific views
- Services that bypass model validations

## Refactoring to Services
- Extract complex model methods
- Move controller logic to services
- Consolidate scattered business logic
- Simplify fat models
- Improve testability

## Documentation Standards
- Document service purpose clearly
- Explain complex business rules
- Provide usage examples
- Note performance considerations
- Include error handling details

## Integration Guidelines
- Call services from controllers
- Use services in background jobs
- Compose services for complex operations
- Keep services framework-agnostic when possible
- Design for future API exposure

## Best Practices Summary
- Keep services focused and single-purpose
- Make dependencies explicit
- Handle errors gracefully
- Write comprehensive tests
- Design for reusability

Remember: Service objects are about organizing complex business logic. Keep them focused, testable, and easy to understand.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/levifig)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/levifig)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
