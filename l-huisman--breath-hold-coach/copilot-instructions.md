## breath-hold-coach

> **BreathHoldCoach** is a mobile app helping women aged 50-75 practice the Deep Inspiration Breath Hold (DIBH) technique for radiation therapy preparation (left-sided breast cancer).

# BreathHoldCoach (BHC) - Project Guidelines

## Project Overview

**BreathHoldCoach** is a mobile app helping women aged 50-75 practice the Deep Inspiration Breath Hold (DIBH) technique for radiation therapy preparation (left-sided breast cancer).

### Medical Context
- DIBH increases lung capacity, protecting heart from radiation exposure
- Patients practice at home after one-time nurse instruction
- App interface is in **Dutch**

---

## Technical Stack

| Category | Technology |
|----------|------------|
| Framework | Expo SDK 54 |
| Language | TypeScript (strict mode) |
| UI | React Native 0.81, React 19 |
| Navigation | Expo Router (file-based) |
| Animations | React Native Reanimated |
| State | React Context API |
| Testing | Jest + React Testing Library |

---

## Development Guidelines

### Prime Directive
- Work on **one file at a time** to avoid corruption
- Explain changes while coding

### TypeScript/React Native Requirements
- Use strict typing, `const`/`let` over `var`
- Arrow functions, async/await, optional chaining (`?.`), nullish coalescing (`??`)
- Destructuring, spread/rest, template literals
- Functional components with hooks

### Accessibility (Required)
- Proper `accessibilityLabel` and `accessibilityRole` on interactive elements
- Adequate color contrast for older users (50-75 age group)
- Clear, readable text sizes
- Support for screen readers

### Error Handling
- Use `try-catch` for async operations
- Provide user-friendly Dutch error messages
- Log technical details for debugging

### File Structure
```
app/                    # Expo Router screens (tabs: index, relax, agenda, messages, profile)
components/             # Reusable UI components
contexts/               # React Context providers
constants/              # App constants (theme, etc.)
hooks/                  # Custom React hooks
```

### Key User Settings
- **Breath Hold Goal**: Target duration (default: 45s)
- **Daily Goal**: Incremental seconds per day
- **Practice Moments**: Scheduled practice times
- **Assistive Learning**: Gradual pacing for users needing it

---

## Commands

| Command | Description |
|---------|-------------|
| `yarn test` | Run tests |
| `yarn lint` | Run ESLint |

> **Important**: Always ask user before installing new dependencies.

---

## Testing (Jest + React Testing Library)

- Write tests for critical features (breath timing, progress tracking)
- Use `@testing-library/react-native` for component tests
- Mock timers with `jest.useFakeTimers()` for breath-hold timing
- Test accessibility with `getByRole` and `getByLabelText`
- Run `yarn test:coverage` to check coverage

---

## React Native Best Practices

### Performance
- Use `FlatList` for long lists, never `ScrollView` with `.map()`
- Memoize callbacks with `useCallback`, values with `useMemo`
- Avoid inline styles; use `StyleSheet.create()`
- Keep components small and focused

### Styling
- Use `StyleSheet.create()` (optimized at build time)
- Use platform extensions (`.ios.tsx`, `.android.tsx`) when needed

### State & Effects
- Minimize re-renders by lifting state only when necessary
- Clean up timers/subscriptions in `useEffect` return
- Use refs for values that shouldn't trigger re-renders

---

## Code Style

### Component Pattern
```tsx
import { View, Text } from 'react-native';

interface Props {
  title: string;
  onPress?: () => void;
}

export const Component = ({ title, onPress }: Props) => {
  return (
    <View accessibilityRole="button" accessibilityLabel={title}>
      <Text>{title}</Text>
    </View>
  );
};
```

### Context Pattern
```tsx
const [state, setState] = useState<Type>(initialValue);
// Use Context API for global state (user settings, progress)
```

### Animation Pattern
```tsx
import Animated, { useSharedValue, withTiming } from 'react-native-reanimated';
// Use Reanimated for smooth breath-hold animations
```

---

## Security & Quality
- Validate all user inputs
- Document complex functions

---
> Source: [l-huisman/Breath-Hold-Coach](https://github.com/l-huisman/Breath-Hold-Coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
