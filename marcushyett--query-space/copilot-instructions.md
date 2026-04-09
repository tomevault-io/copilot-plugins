## query-space

> - **Always write unit tests first** before building logic

# Claude Development Guidelines

## Development Rules

### 1. Test-Driven Development
- **Always write unit tests first** before building logic
- Tests should be placed in `__tests__` directories alongside the code they test
- Use Vitest for all testing
- Ensure tests cover edge cases and error conditions

### 2. Icons - No Emojis
- **Never use emojis** in the codebase
- Use Ant Design icons (`@ant-design/icons`) for all iconography
- Import icons individually to optimize bundle size

### 3. Component Library
- **Use Ant Design modules as much as possible**
- Only use custom styling when absolutely necessary
- Leverage antd's built-in theming system
- Prefer antd components over custom implementations

### 4. Color Palette - Greyscale Only
- **Black and white (dark mode) with greyscale only**
- Primary background: `#000000` (pure black)
- Primary text: `#ffffff` (pure white)
- Secondary/borders: greyscale values (`#1a1a1a`, `#333333`, `#666666`, etc.)
- **Exceptions** (only when there is a strong need):
  - Syntax highlighting in code editors
  - Line differentiation in charts/graphs
  - Data visualization colors
- Normal UI elements (buttons, inputs, borders, etc.) should always be greyscale

### 5. Global CSS Only
- **Always use global CSS** (`app/globals.css`)
- Do not use inline styles or CSS-in-JS for individual components
- Do not use CSS modules
- All styling overrides should be in globals.css with appropriate selectors

### 6. Component Architecture
- **Build small, discrete, testable, and highly maintainable components**
- Prefer many smaller components with smaller files over large monolithic components
- Each component should have a single responsibility
- Extract reusable logic into custom hooks
- Keep component files under 150 lines when possible
- Name components clearly to describe their purpose

### 7. TypeScript Strictness
- **Always check TypeScript errors and fix them properly before committing**
- No `any` types unless absolutely unavoidable (and documented)
- No `@ts-ignore` or `@ts-expect-error` comments
- No type assertions (`as`) unless truly necessary
- Enable strict mode in tsconfig.json
- All function parameters and return types should be explicitly typed

### 8. Linting Before Commit
- **Always run `npm run lint` before committing code**
- All lint errors must be fixed before committing
- Use `unknown` instead of `any` for error handling in catch blocks
- Remove unused variables and imports
- Ensure no ESLint warnings or errors exist in the codebase

### 9. Responsive Design
- **Use Ant Design's native responsive features** as much as possible
- Use `Grid.useBreakpoint()` hook for responsive logic in components
- Prefer antd's responsive props (e.g., `Space wrap`, `maxTagCount="responsive"`) over custom CSS
- **Avoid duplicating code for mobile** - one responsive version is always better
- Only use CSS media queries in `globals.css` for layout adjustments that can't be handled by antd
- Breakpoint reference: `xs` (<576px), `sm` (≥576px), `md` (≥768px), `lg` (≥992px), `xl` (≥1200px)
- Use `!screens.md` to detect mobile devices (screens smaller than 768px)

## File Organization

```
component/
├── ComponentName.tsx           # Main component file
├── __tests__/
│   └── ComponentName.test.tsx  # Tests for component
```

## Commit Checklist

Before committing, ensure:
- [ ] All new code has tests written first
- [ ] No emojis used (antd icons only)
- [ ] Using antd components where possible
- [ ] Colors are greyscale (except for justified exceptions)
- [ ] No inline or component-specific styling
- [ ] Components are small and focused
- [ ] Responsive design uses antd's native features (useBreakpoint, responsive props)
- [ ] Linting passes with no errors (`npm run lint`)
- [ ] TypeScript compiles with no errors (`npm run build`)
- [ ] All tests pass (`npm test`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcushyett)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/marcushyett)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
