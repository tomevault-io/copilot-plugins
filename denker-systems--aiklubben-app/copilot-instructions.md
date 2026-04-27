## 15-mobile-first-ios

> - **Mode**: Always On

# Mobile-First iOS Development - Expo Go

## Activation

- **Mode**: Always On
- **Description**: Rules to ensure consistent rendering on iOS Expo Go vs web

---

## Platform Priority

```
CRITICAL: This app targets iOS via Expo Go as the PRIMARY platform.
Web is secondary. Always code for iOS behavior first.
```

### Platform Rendering Differences

#### ScrollView Centering

```typescript
// WRONG: justifyContent: 'center' in ScrollView contentContainerStyle
// This does NOT center vertically on iOS because the content container
// only grows to fit its content, not the full ScrollView height.
<ScrollView contentContainerStyle={{ justifyContent: 'center' }}>
  {/* Content will NOT be centered on iOS */}
</ScrollView>

// CORRECT: Add flexGrow: 1 so content container fills ScrollView
<ScrollView contentContainerStyle={{ flexGrow: 1, justifyContent: 'center' }}>
  {/* Content IS centered on iOS */}
</ScrollView>

// CORRECT: Use a plain View with flex: 1 when scrolling is not needed
<View style={{ flex: 1, justifyContent: 'center' }}>
  {/* Content IS centered on iOS */}
</View>
```

#### Pressable Style Functions (CRITICAL)

```
NativeWind 4 with jsxImportSource: 'nativewind' intercepts ALL JSX.
Pressable's function-based style={({ pressed }) => [...]} loses
backgroundColor, borderColor, padding etc. on iOS native.

ALWAYS use the View-wrapper pattern for interactive elements.
```

```typescript
// WRONG: Visual styles on Pressable function style - BROKEN on iOS
<Pressable
  style={({ pressed }) => [
    styles.card,          // backgroundColor, border, padding - ALL LOST on iOS
    pressed && styles.cardPressed,
  ]}
>
  <Text>Content</Text>
</Pressable>

// CORRECT: View handles visuals, Pressable handles interaction only
<View style={[styles.card, isSelected && styles.cardSelected]}>
  <Pressable
    style={({ pressed }) => ({
      flex: 1,
      opacity: pressed ? 0.8 : 1,
    })}
  >
    <Text>Content</Text>
  </Pressable>
</View>
```

**Why "Fortsätt →" works but quiz buttons don't:**

- LinearGradient/View with static styles → WORKS on iOS
- Pressable with function `({ pressed }) => [styles.x]` → BROKEN on iOS

**Use StyledPressable component** (`@/components/ui/StyledPressable`) for simple cases.
For complex conditional styles, use the manual View-wrapper pattern.

#### Image Rendering

```typescript
// Images may render differently on iOS vs web.
// ALWAYS specify both width and height explicitly.
// ALWAYS use resizeMode prop.

// CORRECT
<Image
  source={require('./assets/logo.png')}
  style={{ width: 200, height: 200 }}
  resizeMode="contain"
/>

// WRONG: Missing explicit dimensions
<Image
  source={require('./assets/logo.png')}
  style={{ width: '100%' }}
/>
```

---

## Style Rules for Cross-Platform Consistency

### Use StyleSheet.create or Inline Style Objects

```typescript
// PREFERRED: StyleSheet.create for static styles (optimized on iOS)
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#0C0A17',
  },
});

// OK: Inline style objects for dynamic styles
<View style={{ paddingHorizontal: 32 }}>
```

### Avoid NativeWind className for Layout-Critical Styles

```
IMPORTANT: NativeWind/Tailwind className can produce different results
on iOS vs web for layout properties (flex, padding, margin, position).
Use inline styles or StyleSheet for layout-critical properties.

Use className only for simple, non-layout styling (text color, opacity).
```

### Platform.select for Platform-Specific Values

```typescript
import { Platform } from 'react-native';

// Use when iOS and web need different values
const fontSize = Platform.select({
  ios: 17, // iOS prefers 17pt for body text
  default: 16, // Web/Android
});

// Use 'native' key to target both iOS and Android
const shadow = Platform.select({
  native: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  default: {
    boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
  },
});
```

---

## Testing Checklist

```
Before considering any UI change complete:
1. Test on iOS Expo Go FIRST (primary target)
2. Verify centering and alignment on iOS
3. Check touch targets are 44x44pt minimum on iOS
4. Verify animations render on iOS
5. Then check web as secondary
```

---

## Common iOS vs Web Gotchas

1. **flex: 1** - Works identically, but parent MUST also have flex: 1 on iOS
2. **overflow: 'hidden'** - Required on iOS for borderRadius to clip children
3. **transform** - On iOS, transform origin is center; on web it may differ
4. **percentage widths/heights** - Parent MUST have explicit dimensions on iOS
5. **Text wrapping** - Behaves differently; always test with long text on iOS
6. **Shadows** - iOS uses shadowColor/shadowOffset/shadowOpacity; web uses boxShadow
7. **Fonts** - System fonts render differently; always check visual weight on iOS
8. **ScrollView** - contentContainerStyle needs flexGrow: 1 for centering on iOS

---

## Forbidden Practices

1. **NEVER** rely on web browser testing alone - always verify on iOS Expo Go
2. **NEVER** use justifyContent: 'center' in ScrollView without flexGrow: 1
3. **NEVER** use percentage dimensions without explicit parent dimensions on iOS
4. **NEVER** compute complex styles inside Pressable style functions
5. **NEVER** assume NativeWind className produces identical results on iOS and web
6. **NEVER** use web-only CSS properties (boxShadow, cursor, etc.) without Platform.select
7. **NEVER** use string percentages for width/height in nested flex layouts on iOS

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
