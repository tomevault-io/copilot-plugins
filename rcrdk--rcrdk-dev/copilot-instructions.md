## control-flow

> Early returns, single-line statements, and comment preservation


# Control Flow

### Early Returns

Prefer early returns over `else` statements to reduce nesting and improve readability.

- ✅ Good:

  ```typescript
  function processUser(user: User | null) {
    if (!user) return null
    if (!user.isActive) return null

    const upperCaseName = user.name.toUpperCase()
    return upperCaseName
  }
  ```

- ❌ Bad:
  ```typescript
  function processUser(user: User | null) {
    if (user) {
      if (user.isActive) {
        return user.name.toUpperCase()
      } else {
        return null
      }
    } else {
      return null
    }
  }
  ```

### Single-Line Statements

Avoid curly brackets for single-line `if` statements or arrow functions.

- ✅ Good:

  ```typescript
  if (isValid) return true

  const doubled = numbers.map((n) => n * 2)

  const handleClick = () => setCount(count + 1)
  ```

- ❌ Bad:

  ```typescript
  if (isValid) {
    return true
  }

  const doubled = numbers.map((n) => {
    return n * 2
  })

  const handleClick = () => {
    setCount(count + 1)
  }
  ```

## Comments

- Only add comments to code when explicitly asked. Always preserve:
  - "TODO" comments
  - "Tech Debt" comments
  - ESLint disable/enable rules
  - Prettier ignore rules
  - TypeScript `@ts-expect-error` or `@ts-ignore` directives

---
> Source: [rcrdk/rcrdk.dev](https://github.com/rcrdk/rcrdk.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
