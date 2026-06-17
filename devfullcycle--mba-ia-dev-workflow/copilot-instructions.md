## testing

> Use when writing or reviewing tests. Covers test structure (AAA pattern), unit/integration/e2e placement, database cleanup, and test data setup.


## General

- Run only related tests during development; run the full suite before finishing
- All test commands run inside the container: `docker compose exec api npm run test:watch` or `docker compose exec api npx vitest run src/path/to/file.test.ts`

## Test Data

- Use factories or builders to create test data objects
- Avoid hardcoding values in tests; use variables or helper functions to generate test data
- Use realistic data that reflects actual use cases to catch edge cases and ensure test reliability
- Clean up test data after each test to maintain isolation and prevent side effects

## Test Structure

- Follow the Arrange-Act-Assert (AAA) pattern in test cases:
  - **Arrange:** Set up the necessary preconditions and inputs
  - **Act:** Execute the code being tested
  - **Assert:** Verify that the outcome is as expected
- Use descriptive test names that clearly indicate the expected behavior being tested
- Group related tests together using `describe()` blocks for better organization and readability
- Avoid testing multiple behaviors in a single test case; each test should focus on one specific aspect
- Use `beforeAll` and `afterAll` for setup and teardown that applies to all tests in a suite, and `beforeEach` and `afterEach` for setup and teardown that applies to individual tests

## Unit Tests (`*.test.ts`, `*.spec.ts`)

- Place next to the source file in `<module>/__tests__/unit/`
- Mock repository port interfaces with `vi.fn()` — never mock configured libs (JWT, argon2)
- `describe('ClassName')` with descriptive `it('should ...')` blocks

## Integration Tests (`*.integration.test.ts`)

- Place in `<module>/__tests__/integration/`
- Use real PostgreSQL — connect to the Docker `db` service with a test database
- Clean tables with `dataSource.query('TRUNCATE TABLE ... RESTART IDENTITY CASCADE')` in `afterEach`
- Always `await dataSource.destroy()` in `afterAll` to close connections

## E2E Tests (`*.e2e.test.ts`)

- Place in `<module>/__tests__/e2e/`
- Use `supertest` against the Express `app` instance (not the listening server)
- Test complete request/response cycles: status codes, response shape, error cases
- Test authentication and authorization flows (valid token, invalid token, missing token, wrong role)

---
> Source: [devfullcycle/mba-ia-dev-workflow](https://github.com/devfullcycle/mba-ia-dev-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
