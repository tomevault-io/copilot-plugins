## 04-styling-system

> - **Mode**: Always On

# Styling System Rules

## Activation

- **Mode**: Always On
- **Description**: Consistent styling patterns for React Native iOS apps

---

## StyleSheet Rules

### Always Use StyleSheet.create

```typescript
// CORRECT: StyleSheet.create for optimization
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#0C0A17',
  },
});

// WRONG: Inline styles (creates new object every render)
<View style={{ flex: 1, backgroundColor: '#0C0A17' }}>

// WRONG: Object literal outside StyleSheet
const wrongStyles = {
  container: {
    flex: 1,
  },
};
```

### Style Composition

```typescript
// CORRECT: Array syntax for style composition
<View style={[styles.base, styles.variant, customStyle]} />

// CORRECT: Conditional styles
<View style={[
  styles.base,
  isActive && styles.active,
  isDisabled && styles.disabled,
]} />

// CORRECT: Dynamic styles with StyleSheet
const getDynamicStyles = (color: string) => StyleSheet.create({
  container: {
    backgroundColor: color,
  },
}).container;
```

---

## Color System

### Brand Colors (Defined in theme.ts)

```typescript
export const brandColors = {
  // Primary
  purple: '#8B5CF6',
  purpleDark: '#7C3AED',
  purpleLight: '#A78BFA',

  // Secondary
  pink: '#EC4899',
  teal: '#14B8A6',

  // Status
  success: '#10B981',
  successDark: '#059669',
  error: '#EF4444',
  errorDark: '#DC2626',
  warning: '#F59E0B',
  info: '#3B82F6',
} as const;
```

### UI Colors (Semantic)

```typescript
export const uiColors = {
  // Backgrounds
  background: {
    primary: '#0C0A17',
    secondary: '#1A1625',
    tertiary: '#252136',
    elevated: '#2D2640',
  },

  // Text
  text: {
    primary: '#F9FAFB',
    secondary: '#9CA3AF',
    tertiary: '#6B7280',
    inverse: '#0C0A17',
  },

  // Borders
  border: {
    default: '#374151',
    subtle: '#1F2937',
    strong: '#4B5563',
  },

  // Cards
  card: {
    background: '#1A1625',
    border: '#252136',
  },
} as const;
```

### Color Usage Rules

```typescript
// CORRECT: Use semantic color tokens
backgroundColor: uiColors.background.primary

// WRONG: Hardcoded hex values
backgroundColor: '#0C0A17'

// Exception: One-off colors in specific components are acceptable
// but should be documented with a comment
backgroundColor: '#8B5CF608', // 8% opacity purple for subtle bg
```

---

## Typography System

### Font Sizes (iOS Standard)

```typescript
export const fontSize = {
  xs: 11, // Caption, footnote
  sm: 13, // Secondary text
  base: 15, // Body text (iOS default)
  md: 17, // Body text (iOS System)
  lg: 20, // Subheadline
  xl: 22, // Title 3
  '2xl': 28, // Title 2
  '3xl': 34, // Title 1, Large Title
  '4xl': 40, // Display
} as const;
```

### Font Weights

```typescript
export const fontWeight = {
  normal: '400' as const,
  medium: '500' as const,
  semibold: '600' as const,
  bold: '700' as const,
  extrabold: '800' as const,
};
```

### Text Component Variants

```typescript
// Define text variants in Text component
export type TextVariant =
  | 'h1' // 34pt Bold
  | 'h2' // 28pt Bold
  | 'h3' // 22pt Semibold
  | 'h4' // 20pt Semibold
  | 'body' // 17pt Normal
  | 'bodyBold' // 17pt Bold
  | 'caption' // 13pt Normal
  | 'small'; // 11pt Normal

const variantStyles: Record<TextVariant, TextStyle> = {
  h1: { fontSize: 34, fontWeight: '700', lineHeight: 41 },
  h2: { fontSize: 28, fontWeight: '700', lineHeight: 34 },
  h3: { fontSize: 22, fontWeight: '600', lineHeight: 28 },
  h4: { fontSize: 20, fontWeight: '600', lineHeight: 25 },
  body: { fontSize: 17, fontWeight: '400', lineHeight: 22 },
  bodyBold: { fontSize: 17, fontWeight: '700', lineHeight: 22 },
  caption: { fontSize: 13, fontWeight: '400', lineHeight: 18 },
  small: { fontSize: 11, fontWeight: '400', lineHeight: 13 },
};
```

---

## Spacing System

### Spacing Scale

```typescript
export const spacing = {
  0: 0,
  1: 4,
  2: 8,
  3: 12,
  4: 16,
  5: 20,
  6: 24,
  7: 28,
  8: 32,
  10: 40,
  12: 48,
  16: 64,
} as const;
```

### Spacing Usage

```typescript
// CORRECT: Use spacing constants
padding: spacing[4], // 16

// WRONG: Magic numbers
padding: 16,

// Acceptable: Calculated values with comment
padding: spacing[4] + 2, // 18 - accounts for border
```

---

## Border Radius System

### Border Radius Scale

```typescript
export const borderRadius = {
  none: 0,
  sm: 4,
  md: 8,
  lg: 12,
  xl: 16,
  '2xl': 20,
  '3xl': 24,
  full: 9999, // Circular
} as const;
```

### Button/Card Radius Standards

```typescript
// Buttons
const buttonRadius = {
  sm: borderRadius.md, // 8
  md: borderRadius.lg, // 12
  lg: borderRadius.xl, // 16
};

// Cards
const cardRadius = {
  default: borderRadius.xl, // 16
  large: borderRadius['2xl'], // 20
};

// Circular elements (avatars, icons)
const circularRadius = borderRadius.full; // 9999
```

---

## Shadow System

### iOS Shadows

```typescript
export const shadows = {
  sm: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 2,
  },
  md: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.15,
    shadowRadius: 4,
    elevation: 4,
  },
  lg: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.2,
    shadowRadius: 8,
    elevation: 8,
  },
  xl: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 8 },
    shadowOpacity: 0.25,
    shadowRadius: 16,
    elevation: 16,
  },
} as const;

// Colored shadows for branded elements
export const coloredShadows = {
  purple: {
    shadowColor: '#8B5CF6',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.4,
    shadowRadius: 8,
    elevation: 8,
  },
  success: {
    shadowColor: '#10B981',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.4,
    shadowRadius: 8,
    elevation: 8,
  },
};
```

---

## Opacity System

### Standard Opacities

```typescript
export const opacity = {
  disabled: 0.5,
  pressed: 0.8,
  overlay: 0.6,
  subtle: 0.1,
  medium: 0.3,
} as const;
```

---

## Animation Timing

### Duration Constants

```typescript
export const duration = {
  instant: 100,
  fast: 150,
  normal: 250,
  slow: 350,
  slower: 500,
} as const;
```

---

## Forbidden Styling Practices

1. **NEVER** use inline style objects (always StyleSheet.create)
2. **NEVER** hardcode color values (use color tokens)
3. **NEVER** use magic number spacing (use spacing scale)
4. **NEVER** mix px and pt units (React Native uses dp/pt)
5. **NEVER** use percentage for height without explicit parent
6. **NEVER** use negative margins for layout
7. **NEVER** set opacity below 0.4 for interactive elements
8. **NEVER** use shadows without elevation on Android

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
