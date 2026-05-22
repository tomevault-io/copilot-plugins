## testing-guidelines-for-next-js-boilerplate

> description: Guidelines for testing with Vitest



---
description: Guidelines for testing with Vitest
globs: ["tests/**", "components/**", "lib/**"]
alwaysApply: true
---

# Testing Guidelines for Next.js Boilerplate

## Overview
Ensure robust testing with Vitest for all components, utilities, and Supabase Auth functionality, achieving >80% code coverage.

## Unit Testing
- Use **Vitest** with `@testing-library/react` for unit tests.
- Place tests in `tests/` or alongside components (e.g., `components/button/button.test.tsx`).
- Test all public APIs and edge cases:
  ```typescript
  import { render, screen } from '@testing-library/react'
  import Button from './button'

  describe('Button', () => {
    it('renders with correct label', () => {
      render(<Button label="Click me" />)
      expect(screen.getByText('Click me')).toBeInTheDocument()
    })
  })
  ```

## Supabase Auth Testing
- Test authentication flows (login, signup, session management).
- Mock Supabase client:
  ```typescript
  jest.mock('@supabase/ssr', () => ({
    createBrowserClient: () => ({
      auth: {
        signInWithOAuth: jest.fn().mockResolvedValue({}),
        getUser: jest.fn().mockResolvedValue({ data: { user: null } })
      }
    })
  }))
  ```

## Coverage Requirements
- Achieve >80% branch coverage.
- Run `npm run test:coverage` to verify.

## Best Practices
- Use descriptive test names (e.g., `renders with correct label`).
- Test error states and edge cases.
- Avoid testing implementation details; focus on behavior.

---
> Source: [R3n1er/mysecurepassword](https://github.com/R3n1er/mysecurepassword) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
