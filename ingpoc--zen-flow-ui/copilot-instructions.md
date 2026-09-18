## component-development

> Every component should follow the zen-flow-ui architecture pattern:

# Component Development Standards for Zen Flow UI

## Architecture Principles

### Component Structure
Every component should follow the zen-flow-ui architecture pattern:

```
src/components/ui/ComponentName/
├── index.ts              # Export barrel
├── ComponentName.tsx     # Main component
├── ComponentName.test.tsx # Unit tests
├── ComponentName.stories.tsx # Storybook stories (future)
└── types.ts              # Component-specific types
```

### File Organization Guidelines
- **Main Component**: Business logic and rendering
- **Types**: Interface definitions and type exports
- **Tests**: Unit tests with accessibility checks
- **Index**: Clean export interface

## Component Development Template

### Base Component Structure
```tsx
// src/components/ui/ComponentName/ComponentName.tsx
import React, { forwardRef } from 'react';
import { cn } from '@/lib/utils';
import { cva, type VariantProps } from 'class-variance-authority';
import { useZenAnimation } from '@/hooks/useZenAnimation';

// Component variants using CVA
const componentVariants = cva(
  // Base styles using design tokens
  "relative inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-zen-water focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-zen-paper text-zen-ink hover:bg-zen-cloud",
        primary: "bg-zen-water text-zen-light hover:bg-zen-water/90",
        secondary: "bg-zen-mist text-zen-light hover:bg-zen-mist/90",
        destructive: "bg-zen-accent text-zen-light hover:bg-zen-accent/90",
        outline: "border border-zen-cloud bg-transparent hover:bg-zen-cloud hover:text-zen-ink",
        ghost: "hover:bg-zen-cloud hover:text-zen-ink",
        link: "text-zen-water underline-offset-4 hover:underline",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);

// Component interface
interface ComponentNameProps
  extends React.HTMLAttributes<HTMLElement>,
    VariantProps<typeof componentVariants> {
  // Custom props
  asChild?: boolean;
  loading?: boolean;
}

// Main component with forwardRef for proper ref handling
const ComponentName = forwardRef<HTMLElement, ComponentNameProps>(
  ({ className, variant, size, asChild = false, loading, children, ...props }, ref) => {
    const { animate } = useZenAnimation();
    
    // Component logic here
    
    return (
      <element
        ref={ref}
        className={cn(componentVariants({ variant, size, className }))}
        {...props}
      >
        {children}
      </element>
    );
  }
);

ComponentName.displayName = "ComponentName";

export { ComponentName, type ComponentNameProps };
```

### Type Definitions
```tsx
// src/components/ui/ComponentName/types.ts
import type { VariantProps } from 'class-variance-authority';
import type { componentVariants } from './ComponentName';

export interface ComponentNameProps
  extends React.HTMLAttributes<HTMLElement>,
    VariantProps<typeof componentVariants> {
  asChild?: boolean;
  loading?: boolean;
}

export type ComponentNameVariant = VariantProps<typeof componentVariants>['variant'];
export type ComponentNameSize = VariantProps<typeof componentVariants>['size'];
```

### Export Barrel
```tsx
// src/components/ui/ComponentName/index.ts
export { ComponentName, type ComponentNameProps } from './ComponentName';
export type { ComponentNameVariant, ComponentNameSize } from './types';
```

## Code Quality Standards

### TypeScript Requirements
- **Strict Mode**: All components must compile with TypeScript strict mode
- **Proper Interfaces**: Use proper interface inheritance from HTML attributes
- **Generic Support**: Support generic props where appropriate
- **No Any Types**: Avoid `any` types, use proper typing

### Accessibility Requirements
- **ARIA Labels**: All interactive elements need proper ARIA attributes
- **Keyboard Navigation**: Full keyboard support for all interactions
- **Focus Management**: Proper focus indicators and management
- **Screen Reader Support**: Semantic HTML and ARIA descriptions
- **Color Contrast**: Minimum 4.5:1 contrast ratio

### Performance Requirements
- **No Inline Styles**: Use CSS classes and design tokens only
- **Memoization**: Use React.memo for expensive components
- **Lazy Loading**: Support lazy loading where appropriate
- **Tree Shaking**: Ensure components are tree-shakable

## Animation Integration

### GSAP Integration Pattern
```tsx
import { useZenAnimation } from '@/hooks/useZenAnimation';
import { ZEN_TIMING, ZEN_EASING } from '@/lib/animations';

const Component = () => {
  const { animate } = useZenAnimation();
  const elementRef = useRef<HTMLElement>(null);
  
  const handleInteraction = () => {
    animate(elementRef.current, {
      scale: 1.02,
      duration: ZEN_TIMING.fast,
      ease: ZEN_EASING.out
    });
  };
  
  return (
    <element
      ref={elementRef}
      onMouseEnter={handleInteraction}
    >
      Content
    </element>
  );
};
```

### Motion Preferences
Always respect user motion preferences:
```tsx
const shouldReduceMotion = useReducedMotion();

const animationProps = shouldReduceMotion 
  ? {} 
  : {
      initial: { opacity: 0, y: 20 },
      animate: { opacity: 1, y: 0 },
      transition: { duration: ZEN_TIMING.normal }
    };
```

## Styling Guidelines

### Class Variance Authority (CVA) Pattern
Use CVA for component variants:
```tsx
const buttonVariants = cva(
  // Base styles - always applied
  "inline-flex items-center justify-center rounded-md font-medium transition-colors",
  {
    variants: {
      variant: {
        default: "bg-zen-paper text-zen-ink hover:bg-zen-cloud",
        primary: "bg-zen-water text-zen-light hover:bg-zen-water/90",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 px-3",
        lg: "h-11 px-8",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);
```

### Design Token Usage
Always use design tokens instead of hardcoded values:
```tsx
// ✅ Good - Using design tokens
className="bg-zen-cloud text-zen-ink border-zen-mist p-zen-md rounded-zen-md"

// ❌ Bad - Hardcoded values
className="bg-gray-100 text-gray-900 border-gray-300 p-4 rounded-lg"
```

### Responsive Design
Use Tailwind's responsive prefixes:
```tsx
className="p-4 md:p-6 lg:p-8 text-sm md:text-base lg:text-lg"
```

## Testing Standards

### Unit Test Template
```tsx
// src/components/ui/ComponentName/ComponentName.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { ComponentName } from './ComponentName';

expect.extend(toHaveNoViolations);

describe('ComponentName', () => {
  it('renders with default props', () => {
    render(<ComponentName>Test Content</ComponentName>);
    expect(screen.getByText('Test Content')).toBeInTheDocument();
  });

  it('applies custom className', () => {
    render(<ComponentName className="custom-class">Content</ComponentName>);
    expect(screen.getByText('Content')).toHaveClass('custom-class');
  });

  it('handles click events', () => {
    const handleClick = jest.fn();
    render(<ComponentName onClick={handleClick}>Click me</ComponentName>);
    
    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('supports keyboard navigation', () => {
    render(<ComponentName>Keyboard</ComponentName>);
    const element = screen.getByText('Keyboard');
    
    element.focus();
    expect(element).toHaveFocus();
    
    fireEvent.keyDown(element, { key: 'Enter' });
    // Test keyboard interaction
  });

  it('meets accessibility standards', async () => {
    const { container } = render(<ComponentName>Accessible</ComponentName>);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  it('respects reduced motion preferences', () => {
    // Mock reduced motion
    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: jest.fn().mockImplementation(query => ({
        matches: query === '(prefers-reduced-motion: reduce)',
        media: query,
        onchange: null,
        addListener: jest.fn(),
        removeListener: jest.fn(),
      })),
    });

    render(<ComponentName>Motion</ComponentName>);
    // Test that animations are disabled
  });
});
```

### Accessibility Testing
Include accessibility tests for every component:
```tsx
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

it('has no accessibility violations', async () => {
  const { container } = render(<Component />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

## Documentation Standards

### JSDoc Comments
```tsx
/**
 * A flexible button component with multiple variants and sizes.
 * 
 * @example
 * ```tsx
 * <Button variant="primary" size="lg" onClick={handleClick}>
 *   Click me
 * </Button>
 * ```
 */
export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = "default", size = "default", ...props }, ref) => {
    // Component implementation
  }
);
```

### README Documentation
Each component should have usage examples in the main README or component-specific documentation.

## Error Handling

### Error Boundaries
Wrap complex components with error boundaries:
```tsx
import { ErrorBoundary } from 'react-error-boundary';

const ComponentWithErrorBoundary = (props) => (
  <ErrorBoundary fallback={<ComponentErrorFallback />}>
    <Component {...props} />
  </ErrorBoundary>
);
```

### Prop Validation
Use TypeScript interfaces for prop validation instead of PropTypes:
```tsx
interface ComponentProps {
  required: string;
  optional?: number;
  callback: (value: string) => void;
}
```

## Performance Optimization

### React.memo Usage
Use React.memo for components that receive stable props:
```tsx
export const OptimizedComponent = React.memo(({ title, children }) => {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
});
```

### Callback Optimization
Use useCallback for event handlers passed to child components:
```tsx
const handleClick = useCallback((id: string) => {
  // Handle click
}, [dependency]);
```

## Component Checklist

Before submitting a new component:
- [ ] Follows zen-flow-ui architecture pattern
- [ ] Uses design tokens consistently
- [ ] Implements proper TypeScript interfaces
- [ ] Includes comprehensive accessibility features
- [ ] Supports keyboard navigation
- [ ] Respects reduced motion preferences
- [ ] Has unit tests with >90% coverage
- [ ] Includes accessibility tests
- [ ] Uses CVA for variant management
- [ ] Properly handles ref forwarding
- [ ] Includes JSDoc documentation
- [ ] Follows naming conventions
- [ ] Is tree-shakable and performant
- [ ] Works with GSAP animation system
- [ ] Passes all linting rules

## Migration & Backward Compatibility

### Version Management
- Use semantic versioning for breaking changes
- Provide migration guides for major updates
- Maintain backward compatibility when possible
- Use deprecation warnings for removed features

### Legacy Support
```tsx
// Support legacy props with deprecation warnings
interface ComponentProps {
  newProp: string;
  /** @deprecated Use newProp instead */
  oldProp?: string;
}

const Component = ({ newProp, oldProp, ...props }) => {
  if (oldProp && process.env.NODE_ENV === 'development') {
    console.warn('oldProp is deprecated, use newProp instead');
  }
  
  const finalProp = newProp || oldProp;
  // Component implementation
};
```

---
> Source: [ingpoc/zen-flow-ui](https://github.com/ingpoc/zen-flow-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
