## cursor-rules

> - Our goal is to write tests that give us confidence in our application's behavior without being brittle.

---
description:
globs:
alwaysApply: true
---

### Testing Philosophy

- Our goal is to write tests that give us confidence in our application's behavior without being brittle.
- We prioritize integration tests for critical user flows over unit tests for simple components.

### Tools

- **Test Runner:** Vitest
- **Component Testing:** React Testing Library (`@testing-library/react`)
- **End-to-End (E2E) Testing:** Playwright

### Guidelines

1. **Queries:** Use `findBy*` for async elements, `getBy*` for elements expected to be present, and `queryBy*` to assert an element's absence.
2. **User Interactions:** ALWAYS use `@testing-library/user-event` for simulating user interactions, as it more closely resembles real browser behavior.
3. **Mocking:** Use Vitest's `vi.mock` for mocking modules and API requests.
4. **Test Coverage:** Aim for a reasonable test coverage (e.g., >70%) for critical business logic, but do not chase 100% coverage vanity.
5. **Test Naming:** Test descriptions should be clear and describe the expected outcome from a user's perspective. (e.g., `it('should display an error message when the form is submitted with an invalid email')`).
6. **React 19 Testing:** Ensure compatibility with React 19 features including automatic batching improvements and enhanced concurrent features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LikeGeohotz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
