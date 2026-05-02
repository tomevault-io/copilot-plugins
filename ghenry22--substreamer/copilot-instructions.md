## routing-and-navigation

> Expo Router navigation patterns and route file conventions


# Routing & Navigation

## Expo Router (File-Based)

Routes live in `src/app/`. The app uses a Stack navigator at the root and a Tab navigator for the main interface.

## Route File Pattern

Route files are **thin wrappers** that import and render screen components:

```tsx
// src/app/(tabs)/index.tsx
import { HomeScreen } from '@/screens/home';

export default function HomeTab() {
  return <HomeScreen />;
}
```

Keep all business logic, data fetching, and UI in `src/screens/` – not in route files.

## Dynamic Routes

Entity detail screens use dynamic segments: `album/[id].tsx`, `artist/[id].tsx`, `playlist/[id].tsx`.

Access params with `useLocalSearchParams()`:

```tsx
const { id } = useLocalSearchParams<{ id: string }>();
```

## Navigation

Use `useRouter()` for programmatic navigation:

```tsx
const router = useRouter();
router.push(`/album/${album.id}`);
router.push({ pathname: '/album-list', params: { type: listType } });
router.replace('/login');
```

## Layout Files

- `app/_layout.tsx` – Root Stack with auth guard, splash screen, and theme-aware header styles.
- `app/(tabs)/_layout.tsx` – Tab navigator with `MiniPlayer` above the tab bar and `SearchableHeader` as custom header.

## Auth Guard

Authentication redirect logic lives in the root `_layout.tsx`:

```tsx
useEffect(() => {
  if (!rehydrated || splashVisible) return;
  if (!isLoggedIn && !onLoginScreen) router.replace('/login');
  else if (isLoggedIn && onLoginScreen) router.replace('/');
}, [rehydrated, isLoggedIn, splashVisible, segments, router]);
```

## Adding New Routes

1. Create a screen component in `src/screens/new-screen.tsx`.
2. Create a route file in `src/app/new-screen.tsx` that imports and renders the screen.
3. Register the route in `app/_layout.tsx` Stack if it needs custom header options.
4. For entity detail routes, use `app/entity/[id].tsx` pattern.

---
> Source: [ghenry22/substreamer](https://github.com/ghenry22/substreamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
