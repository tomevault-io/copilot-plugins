## ui-components

> - **ALWAYS** use shadcn/ui components from [src/components/ui/](mdc:src/components/ui)

# UI Components & Design System (MANDATORY)

## 🎨 **Component Architecture (MUST FOLLOW)**

- **ALWAYS** use shadcn/ui components from [src/components/ui/](mdc:src/components/ui)
- **ALWAYS** follow the existing component patterns in [src/components/](mdc:src/components)
- **NEVER** create custom UI components without following the design system
- **MUST** use `class-variance-authority (cva)` for component variants

## 🧩 **shadcn/ui Usage (CRITICAL)**

Based on [src/components/ui/button.tsx](mdc:src/components/ui/button.tsx):

- **ALWAYS** use the `cn()` utility function for className merging
- **ALWAYS** implement proper variant systems using cva
- **ALWAYS** use Radix UI primitives as the foundation
- **NEVER** override shadcn/ui component styles directly

```typescript
// ✅ CORRECT - Following shadcn/ui patterns
import { cn } from '@/lib/utils';
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2...",
  {
    variants: {
      variant: { default, destructive, outline, secondary, ghost, link },
      size: { default, sm, lg, icon }
    },
    defaultVariants: { variant: "default", size: "default" }
  }
);

// ❌ WRONG - Not following design system
const CustomButton = ({ className, ...props }) => (
  <button className={`bg-blue-500 text-white px-4 py-2 ${className}`} {...props} />
);
```

## 🚫 **FORBIDDEN UI Practices**

- **NEVER** use inline styles
- **NEVER** create components without proper TypeScript interfaces
- **NEVER** bypass the design system for "quick fixes"
- **NEVER** use hardcoded colors or spacing values
- **NEVER** create components that don't follow accessibility guidelines

## ✅ **REQUIRED UI Practices**

- **ALWAYS** use Tailwind CSS classes for styling
- **ALWAYS** implement proper accessibility attributes (aria-labels, roles)
- **ALWAYS** use semantic HTML elements
- **ALWAYS** implement responsive design patterns
- **ALWAYS** use the established color tokens and spacing scale

## 🎯 **Design Tokens**

- **ALWAYS** use CSS custom properties defined in [src/app/globals.css](mdc:src/app/globals.css)
- **ALWAYS** follow the established color palette and spacing scale
- **ALWAYS** implement dark/light mode support
- **ALWAYS** use consistent border radius and shadow values
  description:
  globs:
  alwaysApply: true

---

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
