## react-astro-components

> Follow atomic design principles with this hierarchy:

# React & Astro Component Creation Rules

## Component Architecture & Organization

### Atomic Design Structure
Follow atomic design principles with this hierarchy:
- **Atoms**: Basic components like buttons, inputs, icons (`src/components/buttons/`, `src/components/form/inputs/`)
- **Molecules**: Simple groups of atoms like cards, form fields (`src/components/cards/`, `src/components/form/`)
- **Organisms**: Complex components like sections (`src/components/sections/`)

### Directory Structure
Each component should have its own directory containing:
```
ComponentName/
├── ComponentName.tsx (React) or ComponentName.astro (Astro)
├── ComponentName.module.scss
├── index.ts (for re-exports)
```

Reference: [ContactButton.tsx](mdc:src/components/buttons/contact/ContactButton.tsx), [ExperienceCard.astro](mdc:src/components/cards/experience/ExperienceCard.astro)

## React Components in Astro

### Component File Structure
```typescript
import type { ComponentProps } from "react";
import styles from "./ComponentName.module.scss";
import Icon from "../../../assets/icons/icon.svg?react";

interface ComponentNameProps extends ComponentProps<"element"> {
  // Specific props here
  label: string;
  variant?: "primary" | "secondary";
}

export const ComponentName = ({ 
  label, 
  variant = "primary", 
  className,
  ...rest 
}: ComponentNameProps) => {
  return (
    <element 
      className={`${styles.component} ${styles[variant]} ${className}`} 
      {...rest}
    >
      {/* Component content */}
    </element>
  );
};
```

### TypeScript Interface Patterns
- Extend appropriate HTML element props: `ButtonHTMLAttributes<HTMLButtonElement>`
- Use meaningful prop names with clear types
- Include optional `className?: string` for style composition
- Use union types for variants: `variant?: "primary" | "secondary"`

Reference: [ContactButton.tsx](mdc:src/components/buttons/contact/ContactButton.tsx)

### SVG Icon Integration
Import SVG icons as React components:
```typescript
import IconName from "../../../assets/icons/icon-name.svg?react";

// Usage
<IconName className={styles["icon-class"]} />
```

## Astro Components

### Component File Structure
```astro
---
import type { TypeDefinition } from "../../../types/type-name";
import styles from "./ComponentName.module.scss";

export interface Props {
  item: TypeDefinition;
  className?: string;
  variant?: "default" | "compact";
}

const { item, className, variant = "default" } = Astro.props;
---

<div class={`${styles["component-name"]} ${styles[variant]} ${className}`}>
  <!-- Component content -->
</div>
```

Reference: [ExperienceCard.astro](mdc:src/components/cards/experience/ExperienceCard.astro)

## Styling Guidelines

### SCSS Modules
- Use CSS modules with `.module.scss` extension
- Use kebab-case for class names: `contact-button`, `experience-card`
- Utilize SCSS variables from theme files

Reference theme variables: [_colors.scss](mdc:src/styles/theme/_colors.scss)

### Theme Integration
Import and use theme variables:
```scss
@import "../../styles/theme/colors";

.component {
  background-color: $primary-background;
  color: $primary-text;
  border: 1px solid $border;
}
```

### Responsive Design
Use mixins for consistent responsive behavior:
```scss
@import "../../styles/theme/mixins";

.component {
  @include mobile {
    // Mobile styles
  }
  
  @include tablet {
    // Tablet styles
  }
}
```

## Component Types & Data

### Type Definitions
Create types in `src/types/` directory:
```typescript
export interface ComponentDataI {
  title: string;
  description: string;
  optional?: boolean;
}
```

Reference: [experience.ts](mdc:src/types/experience.ts)

### Constants
Store component constants in `src/consts/`:
```typescript
export const COMPONENT_VARIANTS = {
  PRIMARY: "primary",
  SECONDARY: "secondary",
} as const;
```

## Component Composition

### Index Files
Create index files for clean imports:
```typescript
// src/components/buttons/index.ts
export { ContactButton } from "./contact/ContactButton";
export { ContactIconButton } from "./custom/ContactIconButton";
```

### Import Patterns
Use path aliases for cleaner imports:
```typescript
import { ComponentName } from "@components/category";
import { CONSTANT } from "src/consts/constants";
```

Reference: [ContactSection.tsx](mdc:src/components/sections/contact/ContactSection.tsx)

## Accessibility Guidelines

### Semantic HTML
- Use appropriate semantic elements
- Include `aria-label` for interactive elements
- Ensure keyboard navigation support

### ARIA Attributes
```tsx
<button 
  aria-label="Descriptive label"
  aria-pressed={isPressed}
  role="button"
>
  Content
</button>
```

## Component Lifecycle

### React Hooks
For complex logic, use custom hooks in appropriate directories:
```
src/components/sections/form/hooks/
```

### Event Handling
Use descriptive handler names:
```typescript
const handleButtonClick = () => {
  // Handler logic
};

const onContactPressed = () => {
  // Contact specific logic
};
```

## Performance Considerations

### Component Optimization
- Use React.memo for expensive renders
- Implement proper key props for lists
- Lazy load heavy components

### Bundle Optimization
- Import only needed utilities
- Use dynamic imports for large dependencies
- Optimize SVG imports

## Internationalization (i18n)

### Text Content
Reference i18n patterns for multilingual support:
```typescript
import { languages } from "../../i18n/ui";
```

Reference: [LanguagePicker.tsx](mdc:src/components/language-picker/LanguagePicker.tsx)

## Best Practices Summary

1. **SOLID Principles**: Single responsibility, clear interfaces
2. **DRY Principle**: Reusable components and styles
3. **Atomic Design**: Proper component hierarchy
4. **TypeScript**: Strong typing for all props and data
5. **Accessibility**: Semantic HTML and ARIA attributes
6. **Performance**: Optimized imports and rendering
7. **Maintainability**: Clear naming and documentation

---
> Source: [hmmatus/personal-portfolio-astro](https://github.com/hmmatus/personal-portfolio-astro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
