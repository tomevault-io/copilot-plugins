## 02-ios-layout-system

> - **Mode**: Always On

# iOS Layout System - React Native Flexbox & Positioning

## Activation

- **Mode**: Always On
- **Description**: Layout rules for iOS compatibility in React Native

---

## Flexbox Layout System

### Core Flexbox Rules for iOS

```typescript
// React Native uses Flexbox with flexDirection: 'column' as DEFAULT
// This is OPPOSITE of web CSS (row is default on web)

// CORRECT: Explicit flex direction
const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'column', // Explicit is better
  },
  row: {
    flexDirection: 'row',
    alignItems: 'center',
  },
});
```

### Flex Property Rules

```typescript
// flex: 1 means "take remaining space"
// ALWAYS use flex: 1 on scrollable content containers

// CORRECT
<View style={{ flex: 1 }}>
  <ScrollView style={{ flex: 1 }}>
    {/* Content */}
  </ScrollView>
</View>

// WRONG - ScrollView won't scroll properly
<View>
  <ScrollView>
    {/* Content */}
  </ScrollView>
</View>
```

---

## Position Absolute Rules (CRITICAL)

### The Golden Rule of Position Absolute

```
CRITICAL: Elements with position: 'absolute' do NOT contribute to parent's
layout dimensions. Parent container MUST have explicit width/height.
```

### Correct Absolute Positioning

```typescript
// CORRECT: Parent has explicit dimensions
const styles = StyleSheet.create({
  container: {
    width: 100,
    height: 100, // EXPLICIT HEIGHT
    position: 'relative',
  },
  absoluteChild: {
    position: 'absolute',
    top: 0,
    left: 0,
    width: 50,
    height: 50,
  },
});

// WRONG: Parent has no height - children will overlap/collapse
const wrongStyles = StyleSheet.create({
  container: {
    // NO HEIGHT SPECIFIED - WILL CAUSE iOS LAYOUT ISSUES
    position: 'relative',
  },
  absoluteChild: {
    position: 'absolute',
    top: 0,
    left: 0,
  },
});
```

### When to Use Absolute Positioning

1. **Overlays and modals** - positioned over content
2. **Floating action buttons** - pinned to corners
3. **Progress indicators** - overlaid on content
4. **Badge indicators** - positioned on icons
5. **Background decorations** - behind main content

### When NOT to Use Absolute Positioning

1. **Regular layout flows** - use Flexbox instead
2. **List items** - use FlatList with proper item heights
3. **Stacked content** - use flexDirection: 'column'
4. **Side-by-side content** - use flexDirection: 'row'

---

## Explicit Dimensions (iOS Requirement)

### Always Specify Dimensions for Complex Layouts

```typescript
// CORRECT: Explicit dimensions for 3D button effect
const BUTTON_SIZE = 72;
const BUTTON_DEPTH = 8;

<View style={{
  width: BUTTON_SIZE,
  height: BUTTON_SIZE + BUTTON_DEPTH, // Total height including depth
}}>
  {/* Button layers with absolute positioning */}
</View>

// WRONG: Relying on content to determine size
<View>
  {/* Absolute children - parent will have 0 height */}
</View>
```

### Component Height Calculation

```typescript
// Always calculate total height for components with depth/shadows
const calculateTotalHeight = (
  baseHeight: number,
  depth: number,
  shadowOffset: number = 0,
): number => {
  return baseHeight + depth + shadowOffset;
};

// Usage
const TOTAL_HEIGHT = calculateTotalHeight(72, 8, 4); // 84
```

---

## Safe Area Implementation

### SafeAreaView Usage

```typescript
import { SafeAreaView } from 'react-native-safe-area-context';

// CORRECT: Wrap entire screen
const Screen = () => (
  <SafeAreaView style={{ flex: 1 }} edges={['top', 'bottom']}>
    <Content />
  </SafeAreaView>
);

// CORRECT: Use insets hook for fine control
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const Screen = () => {
  const insets = useSafeAreaInsets();

  return (
    <View style={{
      flex: 1,
      paddingTop: insets.top,
      paddingBottom: insets.bottom,
    }}>
      <Content />
    </View>
  );
};
```

### Safe Area Edge Cases

```typescript
// Modal with bottom sheet - only bottom safe area
<SafeAreaView edges={['bottom']}>
  <BottomSheet />
</SafeAreaView>

// Full screen content behind status bar
<View style={{ flex: 1 }}>
  <StatusBar translucent backgroundColor="transparent" />
  <ImageBackground style={{ flex: 1, paddingTop: insets.top }}>
    {/* Content */}
  </ImageBackground>
</View>
```

---

## ScrollView Layout Rules

### ScrollView Container Requirements

```typescript
// CORRECT: ScrollView inside flex: 1 container
<View style={{ flex: 1 }}>
  <ScrollView
    style={{ flex: 1 }}
    contentContainerStyle={{
      paddingBottom: insets.bottom + 20,
      paddingHorizontal: 16,
    }}
    showsVerticalScrollIndicator={false}
  >
    {/* Content */}
  </ScrollView>
</View>

// WRONG: No flex on parent
<View>
  <ScrollView>
    {/* Will not scroll on iOS */}
  </ScrollView>
</View>
```

### FlatList Performance Rules

```typescript
// CORRECT: Optimized FlatList
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
  removeClippedSubviews={true}
  maxToRenderPerBatch={10}
  windowSize={5}
  initialNumToRender={10}
/>
```

---

## Dimension Constants

### Screen Dimensions

```typescript
import { Dimensions, Platform, StatusBar } from 'react-native';

const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get('window');

// iOS specific status bar height
const STATUS_BAR_HEIGHT = Platform.select({
  ios: 44, // Default for notched iPhones
  android: StatusBar.currentHeight || 24,
  default: 0,
});

// Tab bar height (iOS standard)
const TAB_BAR_HEIGHT = 49;

// Navigation bar height
const NAV_BAR_HEIGHT = 44;
```

---

## Forbidden Layout Practices

1. **NEVER** use position: absolute without explicit parent dimensions
2. **NEVER** omit flex: 1 on ScrollView parent containers
3. **NEVER** use percentage heights without explicit parent height
4. **NEVER** mix fixed and flex dimensions incorrectly
5. **NEVER** ignore safe areas on notched devices
6. **NEVER** use negative margins for layout (use padding instead)
7. **NEVER** rely on content size for containers with absolute children
8. **NEVER** use aspectRatio without at least one explicit dimension

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
