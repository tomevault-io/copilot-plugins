## project-x

> Standards for API design, testing requirements, and verification for the SW Dev Group Dating App backend.

# API Design and Testing Standards

## Description

Standards for API design, testing requirements, and verification for the SW Dev Group Dating App backend.

## Globs

**/controllers/**/*.rb
**/spec/**/*.rb
**/scripts/**/*.zsh

## Rules

### API Design

- Design RESTful APIs following Rails conventions
- Use consistent response formats for all endpoints
- Provide appropriate HTTP status codes for different scenarios
- Validate all user inputs before processing
- Follow proper authentication and authorization checks

### Testing Requirements

1. **Real-world Testing**: All requests must have a curl request test in our test script to verify real-world functionality.

2. **Test Coverage**: Maintain test coverage above 95% for all new code.

3. **Failed Implementation Protocol**: When an implementation attempt fails, do not try again without first explaining what will be done differently in the next attempt.

4. **Clean Failed Attempts**: When trying a new approach after a failed attempt, remove the failed implementation code first to avoid unused code.

5. **Temporary Scripts**: Delete any test/debugging scripts after use to prevent unnecessary code in the codebase.

### Completion Checklist

A feature is considered complete only when it meets ALL of the following criteria:

- All tests pass (RSpec unit tests and API test script)
- New features include appropriate tests (unit, controller, integration)
- Code passes all RuboCop checks with no warnings
- The curl script passes all endpoint tests
- Database migrations are reversible
- No hardcoded sensitive data or credentials 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/freebeiro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
