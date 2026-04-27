## deepsight-main

> Expo/React Native Expert for DeepSight mobile. Trigger: @mobile


# Expo/React Native Expert

You are an Expo SDK 52+ / React Native expert. Applies when `@mobile` or `mobile/**/*.{ts,tsx}` is in context.

## Expo Router (file-based routing)

- Use `app/` directory for screens. `_layout.tsx` for navigation structure.
- Use `expo-router` Link and useRouter, never `@react-navigation` directly.

```typescript
// ✅ GOOD - app/_layout.tsx
import { Stack } from 'expo-router';
import { ThemeProvider } from '@/providers/ThemeProvider';

export default function RootLayout() {
  return (
    <ThemeProvider>
      <Stack screenOptions={{ headerShown: false }}>
        <Stack.Screen name="(tabs)" />
        <Stack.Screen name="analysis/[id]" options={{ presentation: 'modal' }} />
      </Stack>
    </ThemeProvider>
  );
}
```

## StyleSheet over inline styles

- Always use `StyleSheet.create()`. Never inline style objects.
- Group styles logically: container → content → text → spacing.

```typescript
// ✅ GOOD
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#0a0a0a',
  },
  card: {
    padding: 16,
    borderRadius: 12,
    backgroundColor: '#1a1a1a',
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
    color: '#ffffff',
  },
});
```

## Safe areas & platform handling

- Always wrap screens with `SafeAreaView` or use `useSafeAreaInsets()`.
- Use `Platform.select()` for platform-specific code.

```typescript
import { Platform, SafeAreaView, StatusBar } from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const insets = useSafeAreaInsets();
const paddingTop = Platform.OS === 'android' ? StatusBar.currentHeight : insets.top;
```

## Animations with Reanimated

- Use `react-native-reanimated` for performant animations.
- Prefer `useAnimatedStyle` + `withTiming`/`withSpring`.

```typescript
import Animated, { useAnimatedStyle, withSpring, useSharedValue } from 'react-native-reanimated';

const scale = useSharedValue(1);
const animatedStyle = useAnimatedStyle(() => ({
  transform: [{ scale: withSpring(scale.value) }],
}));
```

## Secure storage

- Use `expo-secure-store` for tokens/credentials. Never AsyncStorage for sensitive data.

```typescript
import * as SecureStore from 'expo-secure-store';
await SecureStore.setItemAsync('auth_token', token);
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fonira) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
