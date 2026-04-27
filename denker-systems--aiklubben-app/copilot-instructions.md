## 03-component-architecture

> - **Mode**: Always On

# Component Architecture Rules

## Activation

- **Mode**: Always On
- **Description**: React Native component structure and organization rules

---

## Component File Structure

### Standard Component Template

```typescript
// ComponentName.tsx

import React from 'react';
import { View, StyleSheet, ViewStyle } from 'react-native';
// 1. React imports first
// 2. React Native imports
// 3. Third-party libraries
// 4. Local imports (components, hooks, utils)
// 5. Types/interfaces
// 6. Constants

// ============================================
// TYPES
// ============================================
interface ComponentNameProps {
  /** Required prop description */
  requiredProp: string;
  /** Optional prop with default */
  optionalProp?: number;
  /** Style override */
  style?: ViewStyle;
  /** Callback function */
  onPress?: () => void;
}

// ============================================
// CONSTANTS
// ============================================
const DEFAULT_VALUE = 10;

// ============================================
// COMPONENT
// ============================================
export const ComponentName: React.FC<ComponentNameProps> = ({
  requiredProp,
  optionalProp = DEFAULT_VALUE,
  style,
  onPress,
}) => {
  // 1. Hooks (in consistent order)
  // 2. Derived state/calculations
  // 3. Event handlers
  // 4. Effects
  // 5. Render helpers
  // 6. Return JSX

  return (
    <View style={[styles.container, style]}>
      {/* Content */}
    </View>
  );
};

// ============================================
// STYLES
// ============================================
const styles = StyleSheet.create({
  container: {
    // Styles here
  },
});
```

---

## Component Categories

### 1. UI Components (`src/components/ui/`)

Base-level, reusable UI elements:

```
ui/
├── Button.tsx          # Primary button component
├── Text.tsx            # Typography component
├── Card.tsx            # Card container
├── Input.tsx           # Form input
├── Badge.tsx           # Badge/chip
├── Avatar.tsx          # User avatar
├── Icon.tsx            # Icon wrapper
├── Skeleton.tsx        # Loading skeleton
├── Divider.tsx         # Visual divider
└── index.ts            # Barrel export
```

### 2. Layout Components (`src/components/layout/`)

Screen structure and navigation:

```
layout/
├── ScreenLayout.tsx    # Base screen wrapper
├── Header.tsx          # Screen header
├── TabBar.tsx          # Bottom navigation
├── SafeContainer.tsx   # Safe area wrapper
├── KeyboardAvoid.tsx   # Keyboard avoiding view
└── index.ts
```

### 3. Feature Components (`src/screens/[feature]/components/`)

Feature-specific components:

```
screens/
├── courses/
│   └── components/
│       ├── LessonNode.tsx
│       ├── LessonPath.tsx
│       └── index.ts
├── lessons/
│   └── components/
│       ├── ProgressBar.tsx
│       └── index.ts
```

### 4. Shared Components (`src/components/shared/`)

Cross-feature reusable components:

```
shared/
├── EmptyState.tsx      # Empty state display
├── ErrorBoundary.tsx   # Error handling
├── LoadingState.tsx    # Loading indicator
└── index.ts
```

---

## Component Props Interface Rules

### Required Props Pattern

```typescript
interface ButtonProps {
  // Required props first (no ?)
  label: string;
  onPress: () => void;

  // Optional props after (with ?)
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;

  // Style overrides last
  style?: ViewStyle;
  textStyle?: TextStyle;
}
```

### Props with Defaults

```typescript
// Define defaults as constants
const BUTTON_DEFAULTS = {
  variant: 'primary' as const,
  size: 'md' as const,
  disabled: false,
  loading: false,
};

// Destructure with defaults
export const Button: React.FC<ButtonProps> = ({
  label,
  onPress,
  variant = BUTTON_DEFAULTS.variant,
  size = BUTTON_DEFAULTS.size,
  disabled = BUTTON_DEFAULTS.disabled,
  loading = BUTTON_DEFAULTS.loading,
  style,
  textStyle,
}) => {
  // Implementation
};
```

---

## Component Size Configurations

### Size System Pattern

```typescript
// Define sizes as typed constant object
const SIZES = {
  sm: {
    height: 32,
    paddingHorizontal: 12,
    fontSize: 14,
    iconSize: 16,
  },
  md: {
    height: 44, // iOS minimum
    paddingHorizontal: 16,
    fontSize: 16,
    iconSize: 20,
  },
  lg: {
    height: 56,
    paddingHorizontal: 24,
    fontSize: 18,
    iconSize: 24,
  },
} as const;

type Size = keyof typeof SIZES;

// Usage in component
const Button: React.FC<{ size?: Size }> = ({ size = 'md' }) => {
  const dimensions = SIZES[size];

  return (
    <Pressable style={{
      height: dimensions.height,
      paddingHorizontal: dimensions.paddingHorizontal,
    }}>
      <Text style={{ fontSize: dimensions.fontSize }}>
        Label
      </Text>
    </Pressable>
  );
};
```

---

## Component Export Rules

### Barrel Exports (index.ts)

```typescript
// src/components/ui/index.ts
export { Button } from './Button';
export { Text } from './Text';
export { Card } from './Card';
export { Input } from './Input';

// Also export types if needed externally
export type { ButtonProps } from './Button';
export type { TextVariant } from './Text';
```

### Named Exports Only

```typescript
// CORRECT: Named export
export const MyComponent: React.FC<Props> = () => {};

// WRONG: Default export
export default function MyComponent() {}
```

---

## Component Composition Patterns

### Compound Components

```typescript
// Card with sub-components
interface CardContextValue {
  variant: 'default' | 'elevated';
}

const CardContext = React.createContext<CardContextValue | null>(null);

const Card = ({ children, variant = 'default' }) => (
  <CardContext.Provider value={{ variant }}>
    <View style={styles.card}>
      {children}
    </View>
  </CardContext.Provider>
);

Card.Header = ({ children }) => (
  <View style={styles.header}>{children}</View>
);

Card.Body = ({ children }) => (
  <View style={styles.body}>{children}</View>
);

Card.Footer = ({ children }) => (
  <View style={styles.footer}>{children}</View>
);

// Usage
<Card variant="elevated">
  <Card.Header>Title</Card.Header>
  <Card.Body>Content</Card.Body>
  <Card.Footer>Actions</Card.Footer>
</Card>
```

---

## Performance Rules

### Memoization Rules

```typescript
// Memoize expensive components
export const ExpensiveList = React.memo(({ items }) => {
  // Render logic
});

// Memoize callbacks passed to children
const ParentComponent = () => {
  const handlePress = useCallback(() => {
    // Handler logic
  }, [dependencies]);

  return <ChildComponent onPress={handlePress} />;
};

// Memoize expensive calculations
const Component = ({ data }) => {
  const processedData = useMemo(() => {
    return expensiveCalculation(data);
  }, [data]);

  return <View>{/* Use processedData */}</View>;
};
```

---

## Forbidden Component Practices

1. **NEVER** use default exports for components
2. **NEVER** define styles inline (always use StyleSheet.create)
3. **NEVER** create components inside other components
4. **NEVER** use anonymous functions in render for callbacks
5. **NEVER** mutate props or state directly
6. **NEVER** skip TypeScript types for props
7. **NEVER** use `any` type for props
8. **NEVER** mix business logic with presentation

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
