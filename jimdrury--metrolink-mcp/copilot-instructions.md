## metrolink-mcp

> These rules provides guidance for Unit Testing


# Testing Guide

## Unit Testing with Vitest

- **Use Vitest as the testing framework** for all unit tests
- Write unit tests for utilities, hooks, and pure functions
- **Colocate test files in the same directory as the file being tested**
- **Use `.spec.ts` or `.spec.tsx` suffix** for test files (not `.test.ts`)
- Pattern:
  ```ts
  // src/lib/utils.ts
  export const formatDate = (date: Date) => {
    // implementation
  };
  
  // src/lib/utils.spec.ts (colocated in same directory)
  import { describe, it, expect } from 'vitest';
  import { formatDate } from './utils';
  
  describe('formatDate', () => {
    it('should format date correctly', () => {
      const date = new Date('2024-01-01');
      expect(formatDate(date)).toBe('Jan 1, 2024');
    });
  });
  ```

## Component Testing

- Test component behavior, not implementation details
- Use React Testing Library for component tests
- Test user interactions and outcomes
- **Colocate test files with components using `.spec.tsx` suffix**
- Pattern:
  ```tsx
  // src/components/Button.tsx
  import type { FC } from 'react';
  
  interface ButtonProps {
    onClick: () => void;
  }
  
  const Button: FC<ButtonProps> = ({ onClick, children }) => (
    <button onClick={onClick}>{children}</button>
  );
  
  export default Button;
  
  // src/components/Button.spec.tsx (colocated in same directory)
  import { describe, it, expect, vi } from 'vitest';
  import { render, screen } from '@testing-library/react';
  import userEvent from '@testing-library/user-event';
  import Button from './Button';
  
  describe('Button', () => {
    it('should call onClick when clicked', async () => {
      const handleClick = vi.fn();
      render(<Button onClick={handleClick}>Click me</Button>);
  
      await userEvent.click(screen.getByRole('button'));
      expect(handleClick).toHaveBeenCalledTimes(1);
    });
  });
  ```

## Test File Organization

- Colocate test files with source files in the same directory
- Use `.spec.ts` for TypeScript test files
- Use `.spec.tsx` for React component test files
- Example structure:
  ```
  src/
    ├── lib/
    │   ├── utils.ts
    │   └── utils.spec.ts          # Colocated test
    ├── components/
    │   ├── Button.tsx
    │   └── Button.spec.tsx       # Colocated test
    └── hooks/
        ├── useAuth.ts
        └── useAuth.spec.ts        # Colocated test
  ```

## Testing Best Practices

- Write tests that describe behavior, not implementation
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)
- Mock external dependencies
- Test error cases and edge cases

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimdrury) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
