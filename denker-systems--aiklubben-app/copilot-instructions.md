## 01-ios-ui-fundamentals

> - **Mode**: Always On

# iOS UI Fundamentals - Apple Human Interface Guidelines

## Activation

- **Mode**: Always On
- **Description**: Core iOS UI rules based on Apple HIG 2025-2026

---

## Touch Targets & Interactive Elements

### Minimum Touch Target Size

```
REQUIRED: All interactive elements MUST have minimum touch target of 44x44 points
```

- **Buttons**: Minimum 44x44pt, recommended 50x50pt for primary actions
- **List items**: Minimum height 44pt
- **Form inputs**: Minimum height 44pt
- **Icon buttons**: Icon can be smaller, but touchable area must be 44x44pt
- **Tab bar items**: Minimum 49pt height (iOS standard)

### Touch Target Implementation in React Native

```typescript
// CORRECT: Explicit hitSlop for small icons
<Pressable
  hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
  style={{ width: 24, height: 24 }}
>
  <Icon size={24} />
</Pressable>

// CORRECT: Minimum touchable area
<Pressable style={{ minWidth: 44, minHeight: 44, padding: 10 }}>
  <Text>Action</Text>
</Pressable>

// WRONG: Small touch target without hitSlop
<Pressable style={{ width: 24, height: 24 }}>
  <Icon size={24} />
</Pressable>
```

---

## Spacing & Layout Rules

### Standard Spacing Values (Apple HIG)

```typescript
const SPACING = {
  // Base unit: 8pt
  xs: 4, // Extra small - icon padding
  sm: 8, // Small - between related elements
  md: 16, // Medium - section padding
  lg: 24, // Large - between sections
  xl: 32, // Extra large - major sections
  xxl: 48, // Double extra large - screen margins
} as const;
```

### Safe Areas (CRITICAL for iOS)

```typescript
// ALWAYS use SafeAreaView or useSafeAreaInsets
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const MyScreen = () => {
  const insets = useSafeAreaInsets();

  return (
    <View style={{
      paddingTop: insets.top,
      paddingBottom: insets.bottom,
      paddingLeft: Math.max(insets.left, 16),
      paddingRight: Math.max(insets.right, 16),
    }}>
      {/* Content */}
    </View>
  );
};
```

### Element Spacing Guidelines

- **Between buttons**: Minimum 12pt, recommended 16pt
- **Between form fields**: 16pt
- **Section margins**: 16-24pt horizontal
- **List item padding**: 16pt horizontal, 12pt vertical
- **Card padding**: 16pt all sides
- **Modal margins**: 20pt from screen edges

---

## Visual Feedback Requirements

### Press States (MANDATORY)

Every interactive element MUST provide visual feedback:

```typescript
// CORRECT: Visual press feedback
<Pressable
  style={({ pressed }) => [
    styles.button,
    pressed && styles.buttonPressed,
  ]}
>
  {({ pressed }) => (
    <Text style={[
      styles.buttonText,
      pressed && styles.buttonTextPressed,
    ]}>
      Action
    </Text>
  )}
</Pressable>

const styles = StyleSheet.create({
  button: {
    backgroundColor: '#8B5CF6',
    opacity: 1,
  },
  buttonPressed: {
    opacity: 0.8,
    transform: [{ scale: 0.98 }],
  },
});
```

### Haptic Feedback Requirements

```typescript
import * as Haptics from 'expo-haptics';

// REQUIRED for all button presses
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);

// REQUIRED for success actions
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);

// REQUIRED for error actions
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error);

// REQUIRED for selection changes
Haptics.selectionAsync();
```

---

## Color & Contrast Requirements

### Minimum Contrast Ratios (WCAG AA)

- **Body text**: 4.5:1 minimum contrast ratio
- **Large text (18pt+)**: 3:1 minimum contrast ratio
- **UI components**: 3:1 minimum contrast ratio

### Dark Mode Support (MANDATORY)

```typescript
// ALWAYS support both light and dark modes
const colors = {
  light: {
    background: '#FFFFFF',
    text: '#000000',
    textSecondary: '#666666',
  },
  dark: {
    background: '#0C0A17',
    text: '#F9FAFB',
    textSecondary: '#9CA3AF',
  },
};
```

---

## Response Time Requirements

### Feedback Timing (Apple HIG)

- **Visual feedback**: Within 100ms of interaction
- **Animation start**: Within 100ms
- **Loading indicator**: Show after 300ms if operation not complete
- **Maximum wait without feedback**: 1 second

---

## Forbidden Practices

1. **NEVER** create touch targets smaller than 44x44pt
2. **NEVER** use color alone to convey information
3. **NEVER** disable haptic feedback on interactive elements
4. **NEVER** use custom fonts without fallback to system fonts
5. **NEVER** place critical actions outside thumb reach zone
6. **NEVER** use opacity below 0.4 for interactive elements
7. **NEVER** use fixed pixel values without considering scale
8. **NEVER** ignore safe areas on notched devices

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
