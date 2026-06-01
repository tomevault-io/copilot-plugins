## expo-template

> **Android and web** are the active targets. Skip iOS-specific UI unless explicitly requested.

## Platform Focus

**Android and web** are the active targets. Skip iOS-specific UI unless explicitly requested.

- Platform checks: `process.env.EXPO_OS` (`"android"` | `"web"` | `"ios"`)
- **Android**: drawer + stack, Lucide via `@/components/icon`; test with **Expo Go** (`bun start` → scan QR) or dev build (`bun run android`)
- **Web**: sidebar layout in `_layout.web.tsx`, Radix menus/tooltips, responsive `md:` breakpoints (`bun run web` or `npx agent-browser`)
- Shared: Uniwind tokens, Lucide icons, same routes in `src/app/` — use `*.web.tsx` / `*.android.tsx` when layouts diverge
- Read skills: `android-expo-app`, `expo-web-app`, `mobile-app-design`, `android-forms-and-input`, `uniwind`, `building-native-ui`

Use `bunx expo install` to add dependencies.

When searching Apple docs, replace https://developer.apple.com with https://sosumi.ai to read as markdown. e.g. https://sosumi.ai/documentation/Xcode/configuring-app-groups instead of https://developer.apple.com/documentation/xcode/configuring-app-groups

## Pressable Style Functions

Do NOT use the function form of `style` on `Pressable` (e.g. `style={({ pressed }) => ({ ... })}`). This is not supported when using Uniwind. Instead, use `className` with the `active:` modifier for pressed states (e.g. `className="bg-transparent active:bg-muted"`).

## CSS Variables

Do NOT use CSS variables (e.g. `var(--app-muted)`) directly in inline `style` props. Instead, use Tailwind classes. The design tokens in `global.css` are mapped to Tailwind colors via the `@theme` block, so use classes like `bg-muted`, `bg-accent`, `border-border`, `text-foreground`, etc. For pressed/active states, use `active:bg-muted` on Pressable components via `className`.

## Verification

**Try Expo Go first** for day-to-day Android testing — fastest iteration, no native build required.

```bash
bun install
bun start                # Dev server — scan QR with Expo Go on Android
```

| Target | Command | When |
|--------|---------|------|
| Android (Expo Go) | `bun start` → Expo Go app | Default for UI, navigation, styling |
| Android (dev build) | `bun run android` | Native module not in Expo Go, pre-release QA |
| Web | `bun run web` or `npx agent-browser` | Sidebar, Radix, responsive layout |

Use a **custom dev build** only when Expo Go lacks a dependency (new native module, Uniwind Pro, etc.) or you need production-like behavior. The repo includes `expo-dev-client` for when you do build locally.

iOS (`npx serve-sim`) only when explicitly building for Apple platforms.

## Metadata

Manage Apple App Store metadata and screenshots with `npx eas-cli@latest metadata:pull` and `npx eas-cli@latest metadata:push`.

---
> Source: [Irere123/expo-template](https://github.com/Irere123/expo-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
