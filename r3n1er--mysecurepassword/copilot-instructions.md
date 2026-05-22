## ui-ux-guidelines-nextjs-boilerplate

> description: Guidelines for UI/UX development with Shadcn and Tailwind CSS



---
description: Guidelines for UI/UX development with Shadcn and Tailwind CSS
globs: ["components/**", "app/**"]
alwaysApply: true
---

# UI/UX Guidelines for Next.js Boilerplate

## Overview
Ensure consistent, accessible, and reusable UI components using Shadcn, Tailwind CSS, and Daisy UI, with a focus on WCAG 2.1 compliance and performance.

## Component Development
- Use **Shadcn** for reusable, accessible components (e.g., Button, Form, Card).
- Structure components as:
  ```typescript
  interface ComponentProps {
    // Define props with TypeScript interfaces
  }
  export function ComponentName({ ...props }: ComponentProps) {
    return <div className="...">{/* JSX */}</div>
  }
  ```
- Store components in `components/` with subfolders (e.g., `components/button/index.tsx`).
- Use **camelCase** for variable/function names and **PascalCase** for components/interfaces.
- Avoid inline styles; use Tailwind CSS classes in `className`.

## Styling
- Use **Tailwind CSS** with **Daisy UI** for theming.
- Define styles in `tailwind.config.js`:
  ```javascript
  module.exports = {
    content: ['./src/**/*.{js,ts,jsx,tsx,mdx}'],
    theme: { extend: {} },
    plugins: [require('daisyui')]
  }
  ```
- Ensure responsive design with Tailwind utilities (e.g., `sm:`, `md:`).
- Support light/dark themes via Daisy UI.

## Accessibility (WCAG 2.1)
- Use semantic HTML (e.g., `<button>`, `<nav>`, `<main>`).
- Apply ARIA attributes where needed (e.g., `aria-label`, `aria-hidden`).
- Ensure full keyboard navigation support.
- Test with tools like `eslint-plugin-jsx-a11y` and `axe`.

## Performance
- Use `React.memo` for pure components.
- Implement `React.lazy` and `Suspense` for code splitting.
- Optimize images with WebP format and lazy-loading.

## Testing
- Test components with **Storybook** for visual regression.
- Write unit tests with **Vitest** for each component (coverage > 80%).

---
> Source: [R3n1er/mysecurepassword](https://github.com/R3n1er/mysecurepassword) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
