## cursor-community-2025

> Testing standards and coverage requirements


# Testing Standards

## Coverage Requirements

**Rule**: Critical modules must have >80% test coverage; all tests must pass.

**Targets**:
- Critical paths: >90% coverage
- Business logic: >80% coverage
- Utilities: >70% coverage

## Tools

- `jest --coverage` or `vitest --coverage`
- `nyc` or `c8` for coverage reporting
- Configure coverage thresholds in `jest.config.js` or `vitest.config.ts`

## Test Structure

**Rule**: Use descriptive test names and limit nesting to 4 levels.

**Example**:

```typescript
// ✅ Good test structure
describe('UserService', () => {
  describe('findUserById', () => {
    it('returns user when found', async () => {
      const user = await userService.findUserById('123');
      expect(user).toBeDefined();
      expect(user.id).toBe('123');
    });

    it('throws NotFoundError when user does not exist', async () => {
      await expect(userService.findUserById('999')).rejects.toThrow(
        NotFoundError
      );
    });
  });
});

// ❌ Too much nesting (5+ levels)
describe('UserService', () => {
  describe('findUserById', () => {
    describe('when user exists', () => {
      describe('with valid ID', () => {
        describe('and permissions', () => {
          it('returns user', () => {});
        });
      });
    });
  });
});
```

## Test Coverage Examples

**Rule**: Test all error paths and edge cases.

**Example**:

```typescript
// ❌ Missing edge case tests
describe('calculateTotal', () => {
  it('calculates total correctly', () => {
    expect(calculateTotal([10, 20])).toBe(30);
  });
});

// ✅ Comprehensive coverage
describe('calculateTotal', () => {
  it('calculates total for positive numbers', () => {
    expect(calculateTotal([10, 20])).toBe(30);
  });

  it('handles empty array', () => {
    expect(calculateTotal([])).toBe(0);
  });

  it('handles negative numbers', () => {
    expect(calculateTotal([-10, 20])).toBe(10);
  });

  it('throws error for invalid input', () => {
    expect(() => calculateTotal(null as unknown as number[])).toThrow();
  });
});
```

## Mocking External Dependencies

**Rule**: Mock external dependencies (APIs, databases, file system).

**Example**:

```typescript
// ✅ Mock external API
import { fetchUser } from '@/lib/api';

jest.mock('@/lib/api');

describe('UserService', () => {
  it('handles API errors', async () => {
    (fetchUser as jest.Mock).mockRejectedValue(new Error('Network error'));
    await expect(userService.getUser('123')).rejects.toThrow();
  });
});
```

## Test-Driven Development

**Rule**: Write tests before or alongside code (TDD preferred).

**Workflow**:
1. Write failing test
2. Implement minimal code to pass
3. Refactor while keeping tests green
4. Repeat

## Prohibited Practices

- 🚫 Never skip tests with `.skip()` or `.only()` in committed code
- 🚫 Never nest `describe/it` blocks more than 4 levels
- 🚫 Never test implementation details (test behavior, not internals)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrescardonas7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
