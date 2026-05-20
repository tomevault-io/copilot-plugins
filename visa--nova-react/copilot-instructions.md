## nova-react

> *              © 2026 Visa

<!--
 *              © 2026 Visa
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *         http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 *
 -->
# CLAUDE.md - Visa Product Design System (VPDS) Nova React Library

This document provides guidance for AI coding agents operating in this repository.

## Project Overview

This is the **VISA Product Design System (Nova) React component library** - a pnpm monorepo containing:

- `libs/nova-react/` - The main React component library (Rollup build)
- `apps/workshop/` - Documentation and demo application (Vite)

**Stack:** React 19, TypeScript 5.9+, Vitest, ESLint 9, Prettier, pnpm 9.x

## Build/Lint/Test Commands

### Common Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Build all workspaces
pnpm build:lib        # Build only the component library
pnpm build:docs       # Build only the workshop/docs
pnpm dev              # Run all in dev mode (parallel)
pnpm dev:lib          # Run library in watch mode
pnpm dev:docs         # Run workshop dev server
pnpm lint             # Lint with auto-fix (zero warnings allowed)
pnpm api:json         # Create the json file for design.visa.com usage
```

### Testing

```bash
pnpm test                           # Run all tests
pnpm test:watch                     # Run tests in watch mode
pnpm test:coverage                  # Run tests with coverage report

# Run a single test file
pnpm vitest run libs/nova-react/src/badge/index.test.tsx

# Run tests matching a pattern
pnpm vitest run badge               # Runs all tests with "badge" in path
pnpm vitest run use-debounce        # Runs hook tests

# Run tests for a specific workspace
pnpm vitest run libs/nova-react     # All library tests
pnpm vitest run apps/workshop       # All workshop tests
```

### Pre-push Checks

```bash
pnpm prepush          # Runs lint && build (used by husky pre-push hook)
```

## Code Style Guidelines

### File Headers

Every source file MUST include the Apache 2.0 license header:

```typescript
/**
 *              © 2025 Visa
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * ...full license text...
 **/
```

### Import Organization

1. External packages first (React, clsx, etc.)
2. Type-only imports use `import type { ... }`
3. Relative imports last

```typescript
import cn from 'clsx';
import type { ComponentPropsWithRef, ElementType } from 'react';

import SomeLocalUtil from '../utils';
```

### Component Structure

```typescript
const CSS_PREFIX = 'v-component-name';

export type ComponentProperties<ET extends ElementType = 'div'> = {
  /** JSDoc comment for prop */
  propName?: PropType;
} & ComponentPropsWithRef<ET>;

/**
 * Component description.
 * @docs {@link https://design.visa.com/components/name | See Docs}
 * @vgar TODO
 * @wcag TODO
 */
const Component = <ET extends ElementType = 'div'>({
  propName,
  className,
  tag: Tag = 'div',
  ...remainingProps
}: ComponentProperties<ET>) => (
  <Tag
    className={cn(CSS_PREFIX, propName && `${CSS_PREFIX}-modifier`, className)}
    {...remainingProps}
  />
);

export default Component;

Component.displayName = 'Component';
```

### Hook Structure

```typescript
/**
 * Hook description
 * @related component1, component2
 * @param {Type} paramName - Description
 * @returns {ReturnType} Description
 */
export const useHookName = (params): ReturnType => {
  // implementation
};

export default useHookName;
useHookName.displayName = 'useHookName';
```

### Naming Conventions

- **Components:** PascalCase (e.g., `Badge`, `InputMessage`)
- **Hooks:** camelCase with `use` prefix (e.g., `useDebounce`, `useFocusTrap`)
- **Types:** PascalCase with `Properties` suffix (e.g., `BadgeProperties`)
- **CSS Prefix:** `v-component-name` (lowercase, kebab-case)
- **Folders:** kebab-case matching component/hook name
- **Test Files:** `index.test.tsx` (components) or `index.test.ts` (hooks)

### Formatting (Prettier)

- Single quotes
- 120 character print width
- 2 space indentation
- Trailing commas (ES5 style)
- No arrow parens when possible
- Semicolons required

### TypeScript

- Strict mode enabled
- Use `import type` for type-only imports (`verbatimModuleSyntax: true`)
- Polymorphic components use `ElementType` generic pattern
- All props must have JSDoc comments
- No unused variables or parameters

### Class Names

- Use `clsx` (imported as `cn`) for conditional class composition
- CSS prefix pattern: `v-{component}` for base, `v-{component}-{modifier}` for variants
- User's `className` prop is always appended last

### Testing Conventions

```typescript
import { render } from '@testing-library/react';
import { axe } from 'jest-axe';
import Component from '.';

describe('Component', () => {
  it('should render defaults correctly', async () => {
    const { container } = render(<Component />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();  // Accessibility check
    expect(container).toMatchSnapshot();
  });
  // Test each prop variation
  // Test custom classNames
  // Test custom tags
  // Test prop permeation
});
```

### Commit Messages (Conventional Commits)

Format: `type(scope): description`

- **Types:** feat, fix, docs, test, refactor, chore, style, perf, build, ci
- **Scope:** Component or hook name in kebab-case (optional)
- **Description:** Imperative mood ("add" not "added"), max 72 chars total

## Project Structure

```
libs/nova-react/src/
├── {component-name}/       # Component folder (kebab-case)
│   ├── index.tsx           # Component implementation
│   ├── index.test.tsx      # Component tests
│   ├── meta.json           # Component metadata
│   └── __snapshots__/      # Jest snapshots
├── use-{hook-name}/        # Hook folder
│   ├── index.ts            # Hook implementation
│   ├── index.test.ts       # Hook tests
│   └── meta.json           # Hook metadata
└── index.ts                # Main exports (auto-generated)

apps/workshop/src/
├── examples/               # Component usage examples
│   ├── components/         # Component examples
│   ├── hooks/              # Hook examples
│   └── patterns/           # Complex pattern examples
└── pages/                  # Workshop pages
```

## Key Dependencies

- **React:** 19.x (uses new JSX transform - no React import needed)
- **clsx:** For className composition
- **@testing-library/react:** For component testing
- **jest-axe:** For accessibility testing
- **vitest:** Test runner with jsdom environment

---
> Source: [visa/nova-react](https://github.com/visa/nova-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
