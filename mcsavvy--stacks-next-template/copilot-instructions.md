## ui-components

> This project uses a component library built on Radix UI primitives with CVA (Class Variance Authority) for styling variants.


# UI Components

This project uses a component library built on Radix UI primitives with CVA (Class Variance Authority) for styling variants.

## Component Architecture

### Base Components
- **Button**: [components/ui/button.tsx](mdc:components/ui/button.tsx) - Button with variants
- **Card**: [components/ui/card.tsx](mdc:components/ui/card.tsx) - Card component
- **Input**: [components/ui/input.tsx](mdc:components/ui/input.tsx) - Input component

### Styling System
- **Tailwind CSS**: Utility-first CSS framework
- **CVA**: Class Variance Authority for component variants
- **Radix UI**: Accessible component primitives
- **Lucide React**: Icon library

## Component Patterns

### Button Component
```typescript
// Use CVA for variant styling
const buttonVariants = cva(
  "base-classes",
  {
    variants: {
      variant: {
        default: "default-styles",
        destructive: "destructive-styles",
      },
      size: {
        default: "default-size",
        sm: "small-size",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);
```

### Component Structure
```typescript
// Use proper TypeScript for component props
interface ComponentProps extends VariantProps<typeof componentVariants> {
  children: ReactNode;
  className?: string;
}

export function Component({ 
  variant, 
  size, 
  className,
  children,
  ...props 
}: ComponentProps) {
  return (
    <div 
      className={cn(componentVariants({ variant, size, className }))}
      {...props}
    >
      {children}
    </div>
  );
}
```

## Styling Guidelines

1. **Use CVA for variants** - Don't use conditional classes
2. **Consistent naming** - Use semantic variant names
3. **Accessibility** - Include proper ARIA attributes
4. **Responsive design** - Use Tailwind responsive prefixes
5. **Dark mode** - Include dark mode variants

## Component Composition

```typescript
// Use asChild pattern for composition
<Button asChild>
  <Link href="/path">Link Button</Link>
</Button>
```

## Icon Integration

```typescript
// Use Lucide React icons
import { ChevronDown, User } from "lucide-react";

<Button>
  <User className="mr-2 h-4 w-4" />
  Profile
</Button>
```

## Best Practices

1. **Composition over inheritance** - Build components that compose well
2. **Consistent API** - Use similar prop patterns across components
3. **Accessibility first** - Include proper keyboard navigation and screen reader support
4. **Performance** - Use React.memo for expensive components
5. **Documentation** - Include JSDoc comments for complex components

---
> Source: [Mcsavvy/stacks-next-template](https://github.com/Mcsavvy/stacks-next-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
