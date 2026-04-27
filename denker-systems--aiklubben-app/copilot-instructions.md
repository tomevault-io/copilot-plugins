## 05-animation-rules

> - **Mode**: Always On

# Animation Rules - React Native & Moti

## Activation

- **Mode**: Always On
- **Description**: Animation patterns for smooth iOS performance

---

## Animation Library Usage

### Primary: Moti (Reanimated 2 wrapper)

```typescript
import { MotiView, MotiText, MotiImage } from 'moti';

// Basic animation
<MotiView
  from={{ opacity: 0, scale: 0.9 }}
  animate={{ opacity: 1, scale: 1 }}
  transition={{ type: 'timing', duration: 300 }}
>
  {/* Content */}
</MotiView>
```

### Secondary: React Native Reanimated 2

```typescript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
} from 'react-native-reanimated';

// For complex, gesture-driven animations
const offset = useSharedValue(0);

const animatedStyle = useAnimatedStyle(() => ({
  transform: [{ translateX: offset.value }],
}));
```

---

## Spring Configurations

### Standard Spring Configs

```typescript
export const SPRING_CONFIGS = {
  // Quick, snappy feedback
  snappy: {
    type: 'spring' as const,
    damping: 18,
    stiffness: 200,
  },

  // Smooth, natural motion
  smooth: {
    type: 'spring' as const,
    damping: 20,
    stiffness: 150,
  },

  // Bouncy, playful motion
  bouncy: {
    type: 'spring' as const,
    damping: 10,
    stiffness: 180,
  },

  // Gentle, subtle motion
  gentle: {
    type: 'spring' as const,
    damping: 25,
    stiffness: 100,
  },

  // Stiff, controlled motion
  stiff: {
    type: 'spring' as const,
    damping: 30,
    stiffness: 300,
  },
} as const;
```

### When to Use Each Config

```typescript
// SNAPPY: Button presses, toggles, quick feedback
<MotiView
  animate={{ scale: pressed ? 0.95 : 1 }}
  transition={SPRING_CONFIGS.snappy}
/>

// SMOOTH: Page transitions, modals, sheets
<MotiView
  animate={{ translateY: visible ? 0 : 300 }}
  transition={SPRING_CONFIGS.smooth}
/>

// BOUNCY: Success animations, celebrations, achievements
<MotiView
  animate={{ scale: [1, 1.2, 1] }}
  transition={SPRING_CONFIGS.bouncy}
/>

// GENTLE: Background elements, subtle movements
<MotiView
  animate={{ opacity: visible ? 1 : 0 }}
  transition={SPRING_CONFIGS.gentle}
/>
```

---

## Timing Configurations

### Standard Timing Configs

```typescript
export const TIMING_CONFIGS = {
  fast: {
    type: 'timing' as const,
    duration: 150,
  },
  normal: {
    type: 'timing' as const,
    duration: 250,
  },
  slow: {
    type: 'timing' as const,
    duration: 400,
  },
} as const;
```

---

## Stagger Animations

### List Item Stagger

```typescript
// Standard stagger delay
export const STAGGER_DELAYS = {
  fast: 30,    // Quick lists
  normal: 50,  // Standard lists
  slow: 80,    // Emphasized entry
} as const;

// Usage in list
{items.map((item, index) => (
  <MotiView
    key={item.id}
    from={{ opacity: 0, translateY: 20 }}
    animate={{ opacity: 1, translateY: 0 }}
    transition={{
      ...SPRING_CONFIGS.smooth,
      delay: index * STAGGER_DELAYS.normal,
    }}
  >
    <ListItem {...item} />
  </MotiView>
))}
```

### Stagger with Maximum Delay

```typescript
// Prevent excessive delays for long lists
const getStaggerDelay = (index: number, maxItems: number = 10) => {
  const effectiveIndex = Math.min(index, maxItems);
  return effectiveIndex * STAGGER_DELAYS.normal;
};
```

---

## Entry Animations

### Screen Entry Animation

```typescript
// Standard screen entry
<MotiView
  from={{ opacity: 0, translateY: 30 }}
  animate={{ opacity: 1, translateY: 0 }}
  transition={{ ...SPRING_CONFIGS.smooth, delay: 100 }}
  style={{ flex: 1 }}
>
  <ScreenContent />
</MotiView>
```

### Component Entry Patterns

```typescript
// Fade in
const fadeIn = {
  from: { opacity: 0 },
  animate: { opacity: 1 },
};

// Fade in up
const fadeInUp = {
  from: { opacity: 0, translateY: 20 },
  animate: { opacity: 1, translateY: 0 },
};

// Scale in
const scaleIn = {
  from: { opacity: 0, scale: 0.8 },
  animate: { opacity: 1, scale: 1 },
};

// Slide in from right
const slideInRight = {
  from: { opacity: 0, translateX: 30 },
  animate: { opacity: 1, translateX: 0 },
};
```

---

## Exit Animations

### Exit Animation Pattern

```typescript
import { AnimatePresence } from 'moti';

// Wrap components that need exit animations
<AnimatePresence>
  {visible && (
    <MotiView
      from={{ opacity: 0, scale: 0.9 }}
      animate={{ opacity: 1, scale: 1 }}
      exit={{ opacity: 0, scale: 0.9 }}
      transition={SPRING_CONFIGS.snappy}
    >
      <Content />
    </MotiView>
  )}
</AnimatePresence>
```

---

## Loop Animations

### Pulse Animation

```typescript
// Subtle pulse for attention
<MotiView
  from={{ scale: 1 }}
  animate={{ scale: [1, 1.05, 1] }}
  transition={{
    type: 'timing',
    duration: 2000,
    loop: true,
  }}
/>
```

### Breathing Animation

```typescript
// Gentle breathing effect
<MotiView
  animate={{
    opacity: [0.5, 1, 0.5],
  }}
  transition={{
    type: 'timing',
    duration: 3000,
    loop: true,
  }}
/>
```

---

## Press Animations

### Button Press Pattern

```typescript
const [isPressed, setIsPressed] = useState(false);

<Pressable
  onPressIn={() => setIsPressed(true)}
  onPressOut={() => setIsPressed(false)}
>
  <MotiView
    animate={{
      scale: isPressed ? 0.95 : 1,
      opacity: isPressed ? 0.8 : 1,
    }}
    transition={SPRING_CONFIGS.snappy}
  >
    <ButtonContent />
  </MotiView>
</Pressable>
```

### 3D Button Press (Duolingo-style)

```typescript
// Depth-based press animation
<Pressable>
  {({ pressed }) => (
    <View style={styles.buttonContainer}>
      {/* Bottom layer (shadow) */}
      <View
        style={[
          styles.buttonBottom,
          {
            transform: [{
              translateY: pressed ? DEPTH / 2 : DEPTH
            }],
          },
        ]}
      />
      {/* Top layer (face) */}
      <MotiView
        animate={{
          translateY: pressed ? DEPTH / 2 : 0,
        }}
        transition={{ type: 'timing', duration: 100 }}
        style={styles.buttonFace}
      >
        <ButtonContent />
      </MotiView>
    </View>
  )}
</Pressable>
```

---

## Performance Rules

### Animation Performance Checklist

1. **Use `transform` and `opacity`** - These are GPU-accelerated
2. **Avoid animating layout properties** - width, height, padding cause layout recalc
3. **Use `useNativeDriver: true`** when using Animated API
4. **Limit concurrent animations** - Max 3-4 simultaneous complex animations
5. **Use `removeClippedSubviews`** for animated lists

### What NOT to Animate

```typescript
// AVOID animating these (causes layout thrashing):
// - width/height (use scale instead)
// - padding/margin (use translateX/Y instead)
// - fontSize (use scale instead)
// - borderWidth
// - flexBasis

// PREFER these (GPU accelerated):
// - opacity
// - transform (scale, translateX, translateY, rotate)
```

---

## Accessibility

### Reduced Motion Support

```typescript
import { useReducedMotion } from 'moti';

const MyComponent = () => {
  const reducedMotion = useReducedMotion();

  return (
    <MotiView
      animate={{ opacity: 1, scale: 1 }}
      transition={reducedMotion ? { type: 'timing', duration: 0 } : SPRING_CONFIGS.smooth}
    >
      <Content />
    </MotiView>
  );
};
```

---

## Forbidden Animation Practices

1. **NEVER** animate layout properties (width, height, padding)
2. **NEVER** use animation duration > 500ms for UI feedback
3. **NEVER** use animation duration < 100ms (imperceptible)
4. **NEVER** ignore reduced motion preferences
5. **NEVER** run more than 4 complex animations simultaneously
6. **NEVER** use JavaScript-driven animations for gestures
7. **NEVER** animate without exit animations for removable content
8. **NEVER** use linear easing for UI animations (use spring/ease)

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
