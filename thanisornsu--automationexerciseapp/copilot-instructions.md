## automationexerciseapp

> - Framework: Playwright + TypeScript

# Project Rules for AI Agent

## Project Context
- Framework: Playwright + TypeScript
- Test Structure: Page Object Model (POM)
- Test Organization: Frontend tests in `tests/frontend/`, API tests in `tests/api/`
- Base URL: https://automationexercise.com
- Reporting: Allure Reports with annotations

## Coding Standards

### Page Object Model
- Always use Page Object Model pattern
- Extend BasePage for all page objects (src/pages/BasePage.ts)
- Locators should be readonly properties
- Methods should be async and return Promise<void> or Promise<boolean>
- Use meaningful method names: verify*, click*, fill*, get*, is*

### Test Files
- Test files: tests/[frontend|api]/[module]/[feature].spec.ts
- Use test.describe() to group related tests
- Test naming: TC_MODULE_NNN format (e.g., TC_LOGIN_001)
- Use beforeEach for common setup
- Use test.step() for logical test sections

### Test Data
- Use TestDataFactory from src/fixtures/test-data.ts
- Generate fresh test data for each test
- Use StaticTestData for fixed test data (credentials, etc.)
- Clean up test data after tests (delete accounts, clear cart, etc.)

### Allure Annotations
- Always include Allure annotations in tests:
  - severity: 'critical' | 'high' | 'medium' | 'low'
  - tags: ['frontend' | 'api', 'smoke' | 'regression' | 'security', etc.]
  - epic: Feature area (e.g., 'User Authentication')
  - feature: Specific feature (e.g., 'Login Page')
  - story: User story description

### File Structure Rules
- Page objects: src/pages/[PageName]Page.ts
- Test files: tests/[frontend|api]/[module]/[feature].spec.ts
- Test data: src/fixtures/test-data.ts
- Constants: src/constants/app-constants.ts
- Types: src/types/index.ts

### Exports and Imports
- Export new page objects from src/pages/index.ts
- Add fixtures to src/fixtures/test-fixtures.ts
- Import fixtures: import { test, expect, TestDataFactory } from '../../src/fixtures'
- Import page objects via fixtures, not directly

## Code Quality

### TypeScript
- Use TypeScript strict mode
- Add JSDoc comments for public methods
- Use async/await (no callbacks)
- Use proper types, avoid 'any'

### Error Handling
- Handle errors gracefully with try-catch
- Use Playwright's built-in wait strategies
- Use expect.soft() for non-critical assertions
- Provide meaningful error messages

### Best Practices
- Use data-qa attributes for selectors when available
- Use meaningful variable names
- Follow AAA pattern: Arrange, Act, Assert
- Keep tests independent (no test dependencies)
- Use test isolation (each test should be able to run alone)

## Test Organization

### Frontend Tests
- Location: tests/frontend/[module]/
- Use page objects via fixtures
- Include UI verification steps
- Use visual assertions when appropriate

### API Tests
- Location: tests/api/[module]/
- Use Playwright's APIRequestContext
- Test request/response validation
- Include status code and response body checks

## When Creating New Code

1. Check existing patterns first (login.spec.ts, LoginPage.ts)
2. Follow the same structure and naming conventions
3. Add proper TypeScript types
4. Include Allure annotations
5. Add JSDoc comments
6. Export from appropriate index files
7. Add to fixtures if needed

## When Modifying Existing Code

1. Maintain backward compatibility
2. Follow existing code style
3. Update related tests if needed
4. Update documentation if API changes
5. Keep the same error handling patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thanisornsu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
