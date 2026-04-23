## katsumascore-design-system

> SCSS styling guidelines and ITCSS architecture patterns


# SCSS Styling Guidelines

## ITCSS Architecture

Follow the ITCSS (Inverted Triangle CSS) methodology for organized, scalable stylesheets.

### Layer Order (Import Order)
1. **Foundation** - [src/scss/foundation/](mdc:src/scss/foundation/)
2. **Global** - [src/scss/global/](mdc:src/scss/global/)
3. **Layout** - [src/scss/layout/](mdc:src/scss/layout/)
4. **Object** - [src/scss/object/](mdc:src/scss/object/)
5. **Inc** - [src/scss/inc/](mdc:src/scss/inc/)
6. **Plugins** - [src/scss/plugins/](mdc:src/scss/plugins/)

## SCSS Best Practices

### Import Statements
- Import variables and mixins at the top
- Use path aliases: `@use '../../global/variable/colors' as color;`

### Example SCSS Structure
```scss
@use '../../global/variable/colors' as color;
@use '../../global/variable/fontWeight' as fontWeight;
@use '../../global/mixin/transition';

.c-component {
  // Base styles

  &__element {
    // Element styles
  }

  &--modifier {
    // Modifier styles
  }

  &:hover {
    // Pseudo-class styles
  }
}
```

## BEM Naming Convention

### Class Naming
- **Block**: `.c-button` (component)
- **Element**: `.c-button__label` (part of component)
- **Modifier**: `.c-button--primary` (variant of component)

### Examples
```scss
.c-button {
  // Base button styles

  &__label {
    // Button label styles
  }

  &--primary {
    // Primary button variant
  }

  &--large {
    // Large button variant
  }
}
```

## Variable Usage

### Color Variables
Reference: [src/scss/global/variable/_colors.scss](mdc:src/scss/global/variable/_colors.scss)

```scss
@use '../../global/variable/colors' as color;

.element {
  background-color: color.$primary;
  color: color.$white;
  border-color: color.$gray300;
}
```

### Font Weight Variables
Reference: [src/scss/global/variable/_fontWeight.scss](mdc:src/scss/global/variable/_fontWeight.scss)

## Mixin Usage

### Transition Mixins
Reference: [src/scss/global/mixin/_transition.scss](mdc:src/scss/global/mixin/_transition.scss)

```scss
@use '../../global/mixin/transition';

.element {
  @include all-transition;
  // or specific transitions
  @include transition(background-color, 0.3s, ease);
}
```

### Font Size Mixins
Reference: [src/scss/global/mixin/_fontSize.scss](mdc:src/scss/global/mixin/_fontSize.scss)

## Responsive Design

### Mobile-First Approach
- Write mobile styles first
- Use min-width media queries for larger screens
- Reference: [src/scss/global/mixin/_screens.scss](mdc:src/scss/global/mixin/_screens.scss)

```scss
.element {
  // Mobile styles (default)
  font-size: 14px;

  @media (min-width: 768px) {
    // Tablet styles
    font-size: 16px;
  }

  @media (min-width: 1024px) {
    // Desktop styles
    font-size: 18px;
  }
}
```

## File Organization

### Component SCSS Files
- Location: [src/scss/object/component/](mdc:src/scss/object/component/)
- Naming: `_component-name.scss`
- Import in component index: [src/scss/object/component/index.scss](mdc:src/scss/object/component/index.scss)

### Utility SCSS Files
- Location: [src/scss/object/utility/](mdc:src/scss/object/utility/)
- Naming: `_utility-name.scss`
- Import in utility index: [src/scss/object/utility/index.scss](mdc:src/scss/object/utility/index.scss)

## Performance Considerations

### Optimization
- Minimize nesting depth (max 3-4 levels)
- Use mixins for repeated patterns
- Group related styles with comments

### Example Optimized SCSS
```scss
// Component: Button
@use '../../global/variable/colors' as color;
@use '../../global/mixin/transition';

.c-button {
  // Base styles
  display: inline-block;
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  @include all-transition;

  // Variants
  &--primary {
    background-color: color.$primary;
    color: color.$white;

    &:hover {
      background-color: color.$primaryHover;
    }
  }

  &--secondary {
    background-color: transparent;
    color: color.$primary;
    border: 1px solid color.$primary;
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/katsun0921) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
