## clarity

> Read the exact versioned docs at https://docs.expo.dev/versions/v57.0.0/ before writing any code.

# Expo HAS CHANGED

Read the exact versioned docs at https://docs.expo.dev/versions/v57.0.0/ before writing any code.

# The design system

Every visual value comes from one place: `constants/theme.ts`. Read it before styling anything.

```tsx
import { colors, radius, spacing, springs, type } from '@/constants/theme';
import { useTheme } from '@/hooks/use-theme';
import { GlassSurface, PrimaryButton, SectionHeader, ThemedText } from '@/components/ui';
```

Four rules, in the order they get broken:

1. **No hardcoded visual values.** No hex colors, no `fontSize`, no raw spacing or radius numbers outside `constants/`. A value used twice is a token. A genuine one-off (an optical nudge on a glyph) may stay inline **with a comment saying why** — that is the only exemption, and the audit below counts the rest.
2. **Colors resolve through `useTheme()`**, never `useColorScheme()` plus a local `light/dark` map. That pattern is what gave the app two greens, six card radii, and seven grays.
3. **Text is `<ThemedText variant=… tone=… />`**. Pick a ramp step and an ink tone; `weight` overrides the step's face. Nothing else names a `fontSize`.
4. **Frosted cards are `<GlassSurface>`**, not a hand-rolled `isLiquidGlassAvailable()` branch. Exception: a card whose *child* needs its own glass must render `<GlassSurface style={StyleSheet.absoluteFill} />` as an absolute sibling, because nested glass does not render on iOS 26.

Components import tokens; screens import components. A screen using `spacing` to lay out its own children is fine. A screen defining a button color is drift.

## Extracting a component

Promote a view into `components/ui/` only when all three hold: it appears in two or more screens, it has a nameable role, and its API is smaller than its implementation. Until then it stays where it is. Never wrap a platform component that already carries the design language (`Switch`, stack headers, `@expo/ui` views) just to route it through the system.

## Auditing for drift

The whole point is that these stay near zero. Run from the repo root:

```bash
grep -rEn '#[0-9a-fA-F]{3,8}\b' app components hooks lib --include='*.tsx' --include='*.ts'
grep -rn 'fontSize: *[0-9]' app components --include='*.tsx'
grep -rn 'borderRadius: *[0-9]' app components --include='*.tsx'
grep -rEn '(padding|margin|gap)[A-Za-z]*:\s*-?[0-9]+' app components --include='*.tsx' \
  | grep -vE ':\s*-?(0|2|4|8|12|16|20|24|32|48)\b'
grep -rEn 'shadow(Color|Offset|Opacity|Radius)|elevation:' app components --include='*.tsx'
```

The spacing whitelist is the scale in `constants/spacing.ts` — update both together. Hits should be the handful of commented one-offs; anything else is a value that escaped the theme.

# Typography

All text uses SF Pro Rounded on iOS and web, bundled in `assets/fonts/` and loaded at runtime in `app/_layout.tsx` (Expo Go can't embed fonts at build time; the expo-font config plugin in `app.json` covers dev builds). Android uses Google Sans Flex Rounded: one family with nine weights in `assets/fonts/android/`, linked at build time by the same plugin as an Android font XML resource under the name `GoogleSansFlexRounded`. Those files are static instances of the Google Sans Flex variable font with the ROND axis pinned to 100; regenerate them with fontTools `varLib.instancer` if one changes.

A face is a `fonts.<name>` object from `@/constants/theme` (`regular`, `medium`, `semibold`, `bold`, `heavy`), spread into a style. On iOS it is a single-face `fontFamily` with no weight. On Android it is the family plus a `fontWeight`. Never set a bare `fontWeight` or `fontFamily` string: on iOS a mismatched weight makes the system synthesize it or fall back to the system font, and on Android a bare family name loses its weight. `constants/fonts.ts` and `constants/fonts.android.ts` must keep the same keys. `ThemedText` handles this — its `weight` prop maps to a face:

```tsx
<ThemedText variant="footnote" weight="bold" tone="secondary">…</ThemedText>
```

Reach for `fonts` directly only where `ThemedText` can't go: a `TextInput` or `Animated.Text` (spread the face), or a prop that takes only a family string (pass `fonts.<name>.fontFamily`, and accept that Android renders it at the regular weight).

# Icons: Hugeicons Pro

This project uses Hugeicons Pro (docs: https://hugeicons.com/docs/integrations/react-native/pro). Two style packages are installed:

- `@hugeicons-pro/core-stroke-rounded` — default for most UI
- `@hugeicons-pro/core-solid-rounded` — filled variant (active/selected states)

## Usage

Render icons with the `HugeiconsIcon` component from `@hugeicons/react-native`. Never use emoji, text glyphs, or other icon libraries.

```tsx
import { HugeiconsIcon } from '@hugeicons/react-native';
import { Mic01Icon } from '@hugeicons-pro/core-stroke-rounded';
import { Mic01Icon as Mic01IconSolid } from '@hugeicons-pro/core-solid-rounded';

<HugeiconsIcon icon={Mic01Icon} size={24} color="#000" strokeWidth={1.5} />
```

Props: `icon`, `size` (default 24), `color`, `strokeWidth` (stroke styles only, default 1.5), plus `altIcon`/`showAlt` for toggling between two icons (e.g. stroke ↔ solid). Icon names are the same across style packages — alias imports (`as XIconSolid`) when mixing both.

## Looking up icon names

Do NOT guess icon names — many have numeric suffixes (`Mic01Icon`, `Mic02Icon`, `MicIcon` all exist). Look them up locally; every icon is a file in the installed package:

```bash
ls node_modules/@hugeicons-pro/core-stroke-rounded/dist/types | grep -i <keyword>
```

Example: `ls node_modules/@hugeicons-pro/core-stroke-rounded/dist/types | grep -i micro` → `Microphone01Icon.d.ts`, `Microphone02Icon.d.ts`, etc. Strip the `.d.ts` to get the import name. For visual browsing, search at https://hugeicons.com/icons.

# Convex backend

Backend code lives in `convex/` and runs on Convex, not in the app.

- **Relative imports only inside `convex/`.** The root tsconfig's `@/*` alias typechecks there and then fails at push, because Convex's bundler does not read the app's paths. `bun run typecheck` runs both programs; the `-p convex` pass is what catches it.
- Functions use the object form with explicit `args` and `returns` validators, `.withIndex(...)` never `.filter(...)`, and read the caller through `requireUserId` in `convex/lib.ts`. No function takes a `userId` argument.
- Screens never call `useQuery`/`useMutation` for app data (account deletion in `app/settings.tsx` is the one exception), and `useConvexAuth` / `<Authenticated>` appear only in `components/convex-sync.tsx`. The stores in `services/` stay the synchronous source of truth so the first frame renders offline; Convex feeds them from behind.
- `CLERK_FRONTEND_API_URL` is a Convex deployment env var, set with `bunx convex env set`, and differs between dev and prod. See `.env.example`.

---
> Source: [SchroederNathan/clarity](https://github.com/SchroederNathan/clarity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
