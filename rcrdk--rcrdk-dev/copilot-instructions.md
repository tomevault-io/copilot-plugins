## naming-conventions

> Avoid reserved words; use meaningful names for props and constants


# Naming Conventions

Use this rule when naming props, constants, and variables. Applies to all TypeScript/JavaScript code.

- Avoid using reserved words (JavaScript/TypeScript keywords) when declaring prop and constant names.
- Use meaningful, descriptive names that clearly indicate purpose and intent.

- ✅ Good:

  ```typescript
  interface ButtonProps {
    label: string
    isDisabled: boolean
    onPress: () => void
  }
  const DEFAULT_PAGE_SIZE = 10
  const isLoading = status === 'pending'
  ```

- ❌ Bad:

  ```typescript
  interface ButtonProps {
    class: string
    default: boolean
    delete: () => void
  }
  const x = 10
  const flag = status === 'pending'
  ```

---
> Source: [rcrdk/rcrdk.dev](https://github.com/rcrdk/rcrdk.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
