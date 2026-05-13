## 060-testing-policy

> **Always write tests before implementation. Follow the Red-Green-Refactor cycle:**


# Testing policy (TDD + mocks)

## TDD (Test-Driven Development)

**Always write tests before implementation. Follow the Red-Green-Refactor cycle:**

### Red-Green-Refactor Cycle

1. **RED** - Write a test that defines the requirement
   - Test should fail initially (no implementation exists)
   - Test describes the expected behavior
   - This ensures the test is actually testing something

2. **GREEN** - Write minimal code to pass the test
   - Write only the code necessary to make the test pass
   - Don't over-engineer at this stage
   - Goal is to get to green as quickly as possible

3. **REFACTOR** - Improve code while keeping tests passing
   - Clean up the code
   - Remove duplication
   - Improve readability
   - Ensure all tests still pass

**IMPORTANT:** Never write implementation code before writing tests. This violates TDD principles.

## Test Types

- Use Feature tests for endpoints and integration flows.
- Use Unit tests for pure business logic.
- Mock only external boundaries (e.g., OpenAI, TMDb). Avoid mocking internal repositories/services.
- Prefer custom fakes/test doubles for interfaces.

## Feature Flags in Tests

- **Developer flags (`category: 'experiments'`):** Test by manually enabling the flag in dev/staging. Do NOT enable developer flags in production.
- **Product flags:** Test both enabled/disabled states when the flag affects behavior.
- **After feature deployment:** Remove developer flags and their conditional code from tests.

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
