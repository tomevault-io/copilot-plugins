## universal-dev-rules

> When writing tests, follow these comprehensive strategies:

# Universal Development Rules - Testing Patterns

When writing tests, follow these comprehensive strategies:

## Test Strategy
- Implement the testing pyramid: unit > integration > e2e
- Aim for 80%+ code coverage with meaningful tests
- Use test-driven development (TDD) for complex business logic
- Apply behavior-driven development (BDD) for user-facing features

## Test Structure
- Follow AAA pattern: Arrange, Act, Assert
- Use descriptive test names that explain the scenario
- Group related tests in describe/context blocks
- Keep tests independent and isolated

## Unit Testing
- Test individual functions and methods in isolation
- Mock external dependencies and services
- Focus on edge cases and error conditions
- Use property-based testing for complex algorithms

## Integration Testing
- Test component interactions and data flow
- Use test containers for database testing
- Test API endpoints with realistic payloads
- Verify external service integrations

## End-to-End Testing
- Test critical user journeys and workflows
- Use page object models for UI testing
- Implement visual regression testing
- Test across different browsers and devices

## Performance Testing
- Include load testing for critical paths
- Test memory usage and resource consumption
- Verify response times meet SLA requirements
- Test under various network conditions

## Test Data Management
- Use factories and builders for test data
- Implement data seeding for consistent test environments
- Clean up test data after each test run
- Use realistic but anonymized production data

QC-01-comprehensive-testing-standards.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/XnimrodhunterX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
