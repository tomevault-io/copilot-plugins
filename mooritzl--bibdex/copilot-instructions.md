## bibdex

> npx expo start          # Start Metro bundler

## Commands

```bash
npx expo start          # Start Metro bundler
npx expo run:ios        # Build and run on iOS
npx expo run:android    # Build and run on Android
npx expo install --fix  # Fix dependency version mismatches
```

## Architecture

- `App.js` — root: GestureHandlerRootView → SafeAreaProvider → ActionSheetProvider → NavigationContainer → 3 native tabs
  - **Home** tab → Stack: `HomeOverviewScreen` (dashboard) → `ProfileScreen` (account) → `LibraryDetailScreen` (favorites tap)
  - **BibDex** tab → Stack: `HomeScreen` (list) → `LibraryDetailScreen` (detail + check-in)
  - **BibMap** tab → `MapScreen` (placeholder)
- `src/store/libraryStore.js` — Zustand store; actions: `setUid`, `subscribeToLibraries`, `subscribeToUserProfile`, `checkIn`, `updateProfile`, `signOut`, `deleteAccount`. Both library subscriptions merge in `MainTabs` via `useEffect`.
- `src/screens/HomeOverviewScreen.js` — discovery hero card (large count + progress + 4 stats), nearby check-in card (5 states), favorites, challenges
- `src/components/CheckInModal.js` — animated reward modal (Reanimated 4.x); gem icon + drain bar; first-discovery vs repeat distinction
- `src/screens/ProfileScreen.js` — avatar with gold ring, trophy stats grid (Playfair numbers), leaderboard row, account/legal settings
- `src/screens/HomeScreen.js` — library list with custom dark filter tabs (All/Discovered/Undiscovered) + discovery progress header (FlatList)
- `src/screens/LibraryDetailScreen.js` — rarity gradient hero with glowing gem icon, XP progress bar, check-in button shows `CheckInModal` then navigates back
- `src/screens/MapScreen.js` — MapView with rarity-colored pins; bottom-right control stack (BlurView group: filter + tracking buttons); content-sized filter sheet (transparent modal + BlurView + spring animation)
- `src/components/LibraryCard.js` — discovered: 2px rarity accent bar + glow shadow; undiscovered: dashed icon + dimmed name
- `src/services/mockData.js` — mock challenges data (libraries now come from Firestore; mockData challenges still used until Firebase challenges are wired)
- `src/theme/colors.js` — light and dark palettes + `RARITY` map + `DISPLAY_FONT` constant
- `src/theme/useTheme.js` — `useTheme()` hook; returns the active palette based on `useColorScheme()`
- `src/components/CardView.js` — generic card wrapper; accepts `style` override; does NOT set `overflow: hidden`
- `src/components/HairlineDivider.js` — `height: StyleSheet.hairlineWidth`
- `src/components/PrimaryButton.js` — props: `backgroundColor`, `icon`, `compact`
- `src/components/InlineProgressBar.js` — track View requires `overflow: hidden`
- `src/components/RarityBadge.js` — dot + uppercase text pill; prop: `rarity` (key into `RARITY` map); `small` variant

State: Zustand (`libraryStore`). Both HomeOverviewScreen and HomeScreen read from the same store.
`totalKP`, `totalCheckIns`, `unlockedCount`, `streak` are **stored aggregates** on `users/{uid}` — mutated with Firestore `increment()`, never recalculated. `checkIn()` is async: optimistic local update first, then `writeBatch` (checkIn event log + libraryProgress upsert + user doc increments); rolls back on error.
Firestore collections: `users/{uid}` (profile + aggregates), `users/{uid}/libraryProgress/{libId}` (per-library state), `users/{uid}/checkIns/{id}` (event log for future stats).
XP/level formula: `level = floor(checkIns/5)+1`, `xp = (checkIns%5)*10`, `xpToNext = 50` (constant).
Streak: computed from `lastCheckInDate` string (`YYYY-MM-DD`) on each check-in write.
Check-in from Home: `expo-location` finds nearest library by Haversine distance; one tap triggers `checkIn(id)` + `CheckInModal`. First unlock = +300 KP; repeat check-in = +50 KP.

## Gotchas

- **Tab navigator**: uses `@bottom-tabs/react-navigation` (native tabs), NOT `@react-navigation/bottom-tabs` — the latter has been removed.
- **Stack navigator**: uses `@react-navigation/native-stack` (`createNativeStackNavigator`), NOT `@react-navigation/stack` — the native stack gives iOS's real swipe-to-go-back with no ScrollView conflicts. Requires `react-native-screens` (already installed).
- **Action Sheet**: `useActionSheet()` from `@expo/react-native-action-sheet` requires `ActionSheetProvider` in `App.js` (already wired). Used in `ProfileScreen` for avatar color picker.
- **CardView overflow**: `CardView` does NOT set `overflow: hidden` — set it on an inner wrapper when needed.
- **SegmentedControl**: installed but no longer used — replaced by custom dark filter tabs in `HomeScreen`.
- **Reanimated peer dep**: `react-native-worklets` (0.5.1) is required by Reanimated 4.x — keep it in `package.json`.
- **CheckInModal**: auto-dismisses after 3400 ms. `isFirst` is derived from `kpGained > 100` (first unlock = 300, repeat = 50).
- **Location snapshot**: Before calling `checkIn(id)`, read the live `isUnlocked` from `libraryList.find(l => l.id === lib.id)` — not from stale state.
- **Mock data**: all libraries are in Munich (real lat/lon). iOS Simulator defaults to Apple HQ — use Features → Location in Simulator to test nearby logic.
- **DISPLAY_FONT**: `Georgia` is used for Playfair Display-style headings/numbers. Import from `src/theme/colors.js`.
- **RARITY map**: import `{ RARITY }` from `src/theme/colors.js` — single source of truth for rarity colors/glows/labels.
- **react-native-svg**: installed (15.12.1) and used for custom vector art — `GemIcon`, `MapPinIcon`, `AppIconSVG`. For standard UI icons prefer `@expo/vector-icons/MaterialIcons`.
- **Firebase**: `libraryStore.js` subscribes to `libraries` + `users/{uid}/libraryProgress` (both via `onSnapshot`, merged client-side). `checkIn` writes a batch to 3 paths atomically. Auth fully wired — `App.js` calls `store.setUid` on `onAuthStateChanged`; `MainTabs` starts both subscriptions.
- **deleteAccount subcollections**: `checkIns` subcollection is NOT deleted by the client (Firestore client SDK can't batch-delete unknown doc IDs without a query). A Cloud Function is needed for full cleanup. `libraryProgress` docs are deleted client-side (IDs are known from the store).
- **`firestore.rules`**: lives at project root, deployed via `firebase deploy --only firestore:rules --project bibdex-423dc`. Always read before editing — contains `isAdmin()` (UID `IzjkIhlJeQOZidT8HpIRHByInFz2`) which gates all admin console writes. Never overwrite without preserving it.
- **Onboarding flow**: `App.js` calls `store.checkOnboarding()` (one-time `getDoc`) after auth. Shows `OnboardingScreen` if `onboardingComplete !== true`. Google sign-in users get name pre-filled from `auth.currentUser.displayName`. `completeOnboarding(name)` writes to Firestore and sets `onboardingComplete: true`. Profile photo deferred post-MVP (requires Firebase Storage, paid tier).
- **iOS Simulator log noise**: `CoreHaptics / hapticpatternlibrary.plist` errors and `TextInputUI / Could not find cached accumulator` warnings are Simulator-only bugs — harmless, never appear on real devices, no action needed.
- **Zustand outside React**: call store actions in non-component contexts (e.g., `onAuthStateChanged`) via `useLibraryStore.getState().action()` — the static getter works anywhere, no hook needed.
- **SafeAreaProvider**: `react-native-safe-area-context` must be wrapped at the root (`App.js`) — `useSafeAreaInsets()` crashes without it. Already wired inside `GestureHandlerRootView`.
- **Row text overflow**: any `flexDirection: 'row'` row containing text next to a fixed element (icon, badge) needs `flex: 1` on the text container — otherwise long names overflow the card boundary. Also add `numberOfLines={1}` on the text to truncate with `…`.
- **RarityBadge in rows**: when placing `<RarityBadge>` alongside text in a `justifyContent: 'space-between'` row, give the sibling text container `flex: 1` so the badge stays pinned to the right edge.
- **`shortName` field**: optional field on library documents; if set, used as the primary display name throughout the app (`shortName || name`). Set in admin EditDrawer. Full `name` shown as a small subtitle on `LibraryDetailScreen` when both are present.
- **Screens in multiple stacks**: When a screen needs to be reachable from two different tabs (e.g., `LibraryDetail` from both Home favorites and BibDex list), add it to both stack navigators in `App.js`. Do NOT use cross-tab `navigate('TabName', { screen: '...' })` — it drops the user into the other tab's stack with no back route to where they came from.
- **Firestore Timestamps in nav params**: Timestamps passed via `navigation.navigate` params are serialized to plain `{ seconds, nanoseconds }` objects — `toDate()` is not available on the receiving end. Always read live Timestamp fields from the store (`libraryList.find(...)`) rather than from `route.params`.
- **`unlockedCount` in UI**: Use `libraryList.filter(l => l.isUnlocked).length` for display — `profile.unlockedCount` (Firestore aggregate) can briefly lag after a check-in. The profile aggregate is still the source of truth for Firestore writes and ProfileScreen stats.
- **`@expo/ui` not usable on SDK 54**: `npx expo install @expo/ui` installs `0.2.0-canary-2026*` which references `ExpoSwiftUI.RNHostViewProtocol` — a protocol that doesn't exist in `expo-modules-core@3.0.29`. The entire `ExpoUI` pod fails to compile. Do not attempt until SDK 55+.
- **`Modal presentationStyle="pageSheet"`** maps to `UISheetPresentationController` but always fills most of the screen — no custom detents or `fitToContents` via React Native's built-in `Modal`. For a compact content-sized sheet use `transparent` modal + `View (flex:1, justifyContent:'flex-end')` + `Animated.View` (no height set) + `BlurView (absoluteFill)`.
- **Blur overlay button pattern** (e.g. map control stack): shadow wrapper `View` (borderRadius + shadow props, NO `overflow:hidden`) → inner clip `View` (same borderRadius, `overflow:'hidden'`) → `BlurView` (absoluteFill) + content. Two wrappers required because iOS clips shadows when `overflow:hidden` is set on the same view.
- **`react-native-maps` does not expose `MKUserTrackingButton`/`MKCompassButton`** — `showsMyLocationButton`/`showsCompass` are flag-only props; the actual UIKit views aren't accessible. A custom native module is needed for direct access.

## Dark Mode

Every screen and component must support both light and dark mode. The app follows the system color scheme automatically via `useColorScheme()`.

**Pattern — always use this in every component:**
```js
import { useTheme } from '../theme/useTheme'; // adjust path as needed

const MyComponent = () => {
  const colors = useTheme();
  const styles = useMemo(() => makeStyles(colors), [colors]);
  // ...
};

const makeStyles = (colors) => StyleSheet.create({
  container: { backgroundColor: colors.background },
  // ...
});
```

**Never hardcode neutral colors** in `StyleSheet` or inline styles. Always use a token from `src/theme/colors.js`:

| Token | Light | Dark | Use for |
|---|---|---|---|
| `background` | `#F3E7D1` | `#0F0D0B` | Screen backgrounds |
| `surface` | `#FEF9EE` | `#1C1916` | Cards, sections (elevated above background) |
| `surfaceSecondary` | `#EDD9B5` | `#242018` | Card-within-card layer (filter tabs, inset content) |
| `textPrimary` | `#1A1210` | `#F2EAD8` | Headings, main text |
| `textSecondary` | `#4A3728` | `#8A7E6A` | Descriptions, body text |
| `textMuted` | `#7A5A3A` | `#3D3730` | Secondary info, labels |
| `textFaint` | `#8A6338` | `#5A4A3A` | Hints, counts |
| `textDisabled` | `#A8834E` | `#2A2218` | Locked states, placeholders |
| `border` | `rgba(107,82,48,0.15)` | `rgba(201,168,76,0.1)` | Dividers/separators — faint gold tint in dark mode |
| `tint` | `#7A5E30` | `#C9A84C` | Gold accent — interactive, active states |

**Surface elevation model** (dark mode, collector aesthetic):
- `background` (`#0F0D0B`) — darkest, screen level
- `surface` (`#1C1916`) — cards, sections, modals (above background)
- `surfaceSecondary` (`#242018`) — filter tab backgrounds, inset elements (above surface)
- Card borders use faint rarity-colored `borderColor: ${r.color}38` rather than the `border` token

**Rarity system** — import `{ RARITY }` from `src/theme/colors.js`:
```js
RARITY = {
  common:    { color: '#9BA8B0', glow: '#9BA8B0', label: 'Common' },
  rare:      { color: '#5B8FD4', glow: '#5B8FD4', label: 'Rare' },
  epic:      { color: '#9B6FD4', glow: '#9B6FD4', label: 'Epic' },
  legendary: { color: '#C9A84C', glow: '#C9A84C', label: 'Legendary' },
}
```
Legendary is gold (same as `tint`). Glow effects: `shadowColor: r.color, shadowOpacity: 0.1–0.3, shadowRadius: 8–24`.

**Hardcoded colors that are intentionally kept** (semantic):
- Status: `#4CAF50` (success/discovered), `#F44336` / `#FF3B30` (destructive), `#30D158` (iOS green toggle)
- `rgba(242,234,216,0.07)` — standard dark progress track / separator fill

## Native Headers

All screens use `useLayoutEffect` (not `useEffect` — avoids title flash) to configure native `UINavigationBar`:
```js
useLayoutEffect(() => {
  navigation.setOptions({ title: 'Screen Name' });
}, [navigation]);
```
- **Large titles (iOS Messages style)**: Use `headerLargeTitle: true` per-screen (not in `screenOptions`). Set `headerStyle: {}` on the same screen to clear the opaque background — this lets iOS render its native translucent/blur material. **Do NOT combine with `headerTransparent: true`** — that disables the large title's UIKit scroll coordination entirely (the large title will never appear).
- **`contentInsetAdjustmentBehavior="automatic"`**: Required on the `ScrollView` or `FlatList` of any screen with `headerLargeTitle: true` — iOS uses this to correctly offset content below the large title area.
- **Large title alignment**: iOS positions the large title at 16pt from the left edge. Content containers on large-title screens should use `paddingHorizontal: 16` (not 20) to align.
- **`headerLargeTitleStyle`**: Use font strings directly (e.g. `fontFamily: 'Georgia'`) — importing `DISPLAY_FONT` into `App.js` triggers a Hermes ReferenceError.
- `LibraryDetailScreen` overrides `headerStyle.backgroundColor` with `colors.background` (dark, matches hero)
- `HomeOverviewScreen` adds a `headerRight` avatar button (initials circle with gold ring) via `setOptions`
- `MapScreen` is **headerless** (full-screen map with filter overlay — no `setOptions` call needed)
- `HomeNavigator` + `BibDexNavigator` call `useTheme()` internally to set stack-level `headerStyle`/`headerTintColor`/`headerTitleStyle`

## Rarity System

Libraries have one of four rarities: `common` · `rare` · `epic` · `legendary`.
Always use `import { RARITY } from '../theme/colors'` — never hardcode rarity colors.
KP on first check-in (unlock): +300 KP. Repeat: +50 KP.

## Admin Console

- Lives in `admin/` — Vite + React SPA, completely separate from Metro/Expo.
- `cd admin && npm run dev` — starts on port 5173 (or `--port 5174` to avoid conflicts).
- `cd admin && npm run build` — outputs to `admin/dist` for Firebase Hosting.
- Auth: Google Sign-In popup → UID check (`IzjkIhlJeQOZidT8HpIRHByInFz2`). Do NOT use email-based rules — `email_verified` is unreliable with Google popup auth; UID never changes.
- `firebase deploy --only firestore:rules --project bibdex-423dc` — always include `--project` flag or it errors.
- **Multi-site hosting** (targets in `firebase.json` + `.firebaserc`): target `web` = `web/` (public site, default site `bibdex-423dc` → `bibdex.lenhard.xyz`), target `admin` = `admin/dist` (site `bibdex-admin`, must be created once via `firebase hosting:sites:create bibdex-admin --project bibdex-423dc`).
- `firebase deploy --only hosting:web --project bibdex-423dc` — deploys public site; `--only hosting:admin` deploys the console. Plain `--only hosting` deploys BOTH.

## Public Website (`web/`)

- Static HTML/CSS, no build step — `index.html` (landing), `privacy.html`, `terms.html`, `impressum.html`, shared `styles.css` (mirrors app palette from `src/theme/colors.js`).
- `cleanUrls: true` in `firebase.json` → live URLs are `/privacy`, `/terms`, `/impressum` (no `.html`). Local preview (`python3 -m http.server`) does NOT rewrite — test with `.html` suffix.
- Legal pages have `[Straße Hausnummer]` / `[PLZ]` / `[E-Mail-Adresse]` placeholders that MUST be filled before deploy; Impressum is required in DE (§ 5 DDG).
- Legal pages carry `<meta name="robots" content="noindex">`; landing page is indexable.
- **Library `active` field**: `active !== false` = visible in app (missing field also counts as active). Archive = write `{ active: false }` with merge. `libraryStore.js` filters with `where('active', '!=', false)`.
- **Admin components**: `AuthGate` (Google sign-in / access denied), `Sidebar` (nav + sign-out), `LibraryTable` (search/filter/sort/multi-select), `EditDrawer` (create/edit/archive/delete), `StatsBar` (active-only rarity counts), `Toast`, `Rarity` (shared rarity constants + `RarityDot`/`RarityBadge`).
- **Bulk archive/restore**: `writeBatch` in `LibrariesPage.handleBulkToggleActive` — select rows via checkboxes, action bar appears above table.

## Google Sign-In (iOS)

- Requires an **iOS app** registered in Firebase console (bundle ID: `com.anonymous.bibdex`) — web client alone cannot handle custom URI scheme redirects on native builds.
- `GoogleService-Info.plist` values used:
  - `REVERSED_CLIENT_ID` → `app.json` `CFBundleURLTypes[0].CFBundleURLSchemes[0]`
  - `CLIENT_ID` → `iosClientId` in `Google.useAuthRequest` in `LoginScreen.js`
- Adding/changing the URL scheme requires a native rebuild: `npx expo run:ios`
- `LoginScreen.js` lives at `src/screens/LoginScreen.js` — handles email/password + Google; Apple is a disabled placeholder.

## Adding Dependencies

Always use `npx expo install <pkg>` (not `npm install`) — Expo pins compatible versions automatically.

## Known Issues / TODOs

- [x] `@react-navigation/bottom-tabs` removed
- [x] Firebase: use subpath imports only (`firebase/firestore`, not top-level `firebase`) to enable tree-shaking
- [x] Admin console built — `admin/` Vite app with library CRUD + archive/restore + Google Sign-In
- [x] Streak: computed from `lastCheckInDate` on each check-in, stored on `users/{uid}`
- [x] Profile: `displayName` + `avatarColor` persisted to Firestore; `updateProfile` writes via `setDoc` merge
- [x] Profile: shows `profile.email` from Firebase auth
- [x] Profile → Sign Out: calls `auth.signOut()` via `store.signOut()`
- [x] Profile → Delete Account: calls `auth.currentUser.delete()` + Firestore cleanup via `store.deleteAccount()`; handles `auth/requires-recent-login` with friendly alert
- [x] Onboarding: new users shown `OnboardingScreen` after sign-up — collects name + photo before entering app; `onboardingComplete` flag on `users/{uid}` controls routing in `App.js`
- [ ] Challenges: wire progress to real `libraryList` state (currently static mock values)
- [ ] deleteAccount: Cloud Function needed to fully delete `users/{uid}/checkIns` subcollection
- [ ] Legal: replace placeholder URLs (`bibdex.app/privacy`, `bibdex.app/terms`) with real hosted pages before App Store submission

## Git / Deploy Notes

- Remote uses HTTPS by default; push requires SSH key or PAT — switch with `git remote set-url origin git@github.com:moOritzl/BibDex.git`
- `.gitignore` excludes `.idea/` (JetBrains) and `admin/dist/` (Vite build) — don't re-add these
- Firebase client `apiKey` in `src/services/firebase.js` and `admin/src/firebase.js` is intentionally committed — it's a public identifier, security comes from Firestore rules

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

---
> Source: [moOritzl/BibDex](https://github.com/moOritzl/BibDex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
