## testing

> Rules for writing unit and e2e tests

* If ever in doubt, use context7 mcp or search the web to help write a unit test
* More tests documentation found in [README.md](mdc:tests/README.md).(tests/README.md)

## Working with Vitest

* Run all tests with `nr test`
* Run a single test with `nr test [path-to-test-file]` for example: `nr test tests/unit/users.get.test.ts`
* Always use the `--watch=false` flag when running tests

## Unit Tests
* Unit tests are written with Vitest
* Are written alongside the code they are testing (in the same directory)
* Some helpful utilities:
    - `mountSuspended()` - Mount components with Nuxt context
    - `renderSuspended()` - Render with Testing Library
    - `mockNuxtImport` allows you to mock Nuxt's auto import functionality
    - `mockComponent` allows you to mock Nuxt's component.
    - All Nuxt composables (`useRoute`, `useRouter`, etc.)
    - Auto-imports work as expected

## API Tests
* API tests are written with Vitest
* Are written alongside the code they are testing (in the same directory)
* API tests are named like `[resource-name].get.test.ts`
* Use the `$vitestFetch` global variable to make requests to the API (automatically configured with the correct base URL from .env.test)
* Run all API tests with `nr test server/api`
* Run a single API test with `nr test [path-to-test-file]` for example: `nr test server/api/users.get.test.ts`


## E2E Tests
* E2E tests are written with Playwright
* E2E tests are written in the tests/e2e/ directory
* Some helpful utilities:
    - `goto(path)` - Navigate to pages
    - `$fetch(url)` - Fetch server responses
    - `createPage(url)` - Create browser pages
    - Full Playwright API available 
* E2E Test Example:
```typescript
import { test, expect } from "@nuxt/test-utils/playwright";

test("homepage works", async ({ page, goto }) => {
  await goto("/");
  await expect(page.getByText("Hello World")).toBeVisible();
});
```

## 🎯 Best Practices

### Test Organization

- Use descriptive test file names that match source files
- Group related tests in describe blocks
- Use relative imports in co-located tests

### Unit Tests

- Use the `@vitest-environment nuxt` comment at the top of the file to run tests in the Nuxt environment 
- Test one thing at a time
- Use descriptive test names
- Each test should only test what's described in the test name
- Mock external dependencies
- Test edge cases and error conditions

### API Tests

- These are distinct from unit tests even though they are written with Vitest

### Component Tests

- Test component behavior, not implementation
- Use realistic props and slots
- Test user interactions
- Verify rendered output

### E2E Tests

- Test critical user journeys
- Use data-testid attributes for reliable selectors
- Test responsive behavior
- Include accessibility checks


## 📚 Resources

- [Nuxt Test Utils Documentation](https://nuxt.com/docs/getting-started/testing)
- [Vitest Documentation](https://vitest.dev/)
- [Playwright Documentation](https://playwright.dev/)
- [Vue Test Utils](https://vue-test-utils.vuejs.org/)

---
> Source: [vueschool/aidd-workhop](https://github.com/vueschool/aidd-workhop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
