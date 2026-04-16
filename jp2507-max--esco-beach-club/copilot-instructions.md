## esco-beach-club

> - Keep `className` **stable**. Static styles in `className`, animated/gesture styles via Reanimated `style`.


# Uniwind Styling Guidelines

## Core Principles

- Keep `className` **stable**. Static styles in `className`, animated/gesture styles via Reanimated `style`.
- Prefer explicit `dark:` pairs (e.g., `bg-background dark:bg-dark-bg`) for all themed elements, as CSS variables do not auto-switch modes.
- Never toggle Tailwind classes per frame; derive animation values in worklets.
- Follow `styling-guidelines.md` for Reduced Motion and animation runtime behavior.
- Class order: layout → flex → spacing → size → border → bg → text → effects → dark.
- Custom components must forward/merge `className` via `cn` (tailwind-merge).

---

## UI & Theming (Uniwind + Tailwind CSS v4)

- **CSS runtime**: Uniwind via `@import 'tailwindcss';` and `@import 'uniwind';` in `global.css`, configured with `withUniwindConfig` in `metro.config.js` using a **relative** `cssEntryFile` path.
- **Scan root awareness**: `cssEntryFile` determines Tailwind's scan root; if code lives outside that tree, add `@source` directives in `global.css`.
- **Metro composition**: when composing Metro wrappers, keep `withUniwindConfig` as the outermost wrapper.
- **Color tokens**: defined in `global.css` `@theme` block (CSS-first, no `tailwind.config.js`). JS-side mirror in `constants/colors.ts` (default export) for use outside className.
- **Dark mode**: `userInterfaceStyle: "automatic"` in `app.json`. Uniwind supports both `dark:` variants and CSS-variable theming.
- **Theme styling default**: use standard light/dark pairs (`bg-background dark:bg-dark-bg`) for all shared UI as variables do not auto-switch.

- **useColorScheme**: import from `react-native` when JS logic needs scheme values.
- **Theme providers**: no styling-library ThemeProvider is required; keep React Navigation `ThemeProvider` only for APIs that require JS theme colors (navigation container, headers).
- **Safe area**: handled via `react-native-safe-area-context` using the `useSafeAreaInsets` hook. Uniwind is used only for styling via `withUniwind` / `withUniwindConfig`.

---

## Component Imports (Critical)

Use local wrappers by default for consistency, while allowing direct RN imports when needed:

- **Prefer wrappers**: `@/src/tw`, `@/src/tw/image`, and `@/src/tw/animated` for className-based UI in app code.
- **`react-native` directly**: allowed for APIs not wrapped in `@/src/tw` (e.g., `Modal`, hooks, platform APIs, type imports).
- **Uniwind binding support**: direct RN components can use Uniwind class bindings where supported (`className`, and mapped `*ClassName` props such as `thumbColorClassName` / `trackColorForFalseClassName` / `trackColorForTrueClassName` on `Switch`).
- **`react-native-reanimated` directly**: use for animation hooks/APIs, and for animated components that do not use `className`.
- **Third-party components**: use `withUniwind` when they do not support `className` or do not forward `style` correctly.

---

## Esco Beach Club Color Tokens

For JS values, import `Colors` from `@/constants/colors`.

Standard Light/Dark pairs (match `global.css` `@theme` block):

- App background: `bg-background dark:bg-dark-bg`
- Elevated surface: `bg-card dark:bg-dark-bg-card`
- Elevated container: `bg-white dark:bg-dark-bg-elevated`
- Border: `border-border dark:border-dark-border`
- Border accent: `border-border-light dark:border-dark-border-bright`
- Primary text: `text-text dark:text-text-primary-dark`
- Secondary text: `text-text-secondary dark:text-text-secondary-dark`
- Muted text: `text-text-muted dark:text-text-muted-dark`
- Brand / CTA: `bg-primary dark:bg-primary-bright`, `text-primary dark:text-primary-bright`
- Danger: `bg-danger dark:bg-error-dark`
- Warning: `bg-warning dark:bg-warning-dark`

---

## Component className Forwarding

Custom components **must** forward and merge `className` props:

```tsx
import { cn } from '@/src/lib/utils';

function Card({ className, ...props }: { className?: string }) {
  return (
    <View
      className={cn('rounded-lg bg-card p-4 dark:bg-dark-bg-card', className)}
      {...props}
    />
  );
}
```

---

## ✅ Do / Avoid

**Do**: Prefer token-based theme classes; keep `className` stable; use Tailwind for static styles; forward `className` via `cn`; use tokens from `global.css` `@theme`; import components from `@/src/tw` when using `className`.

**Avoid**: Per-frame class churn; hard-coded hex colors in className; bypassing shared wrappers without a concrete reason.

---

## Motion Boundary

- Uniwind owns static layout and theme styling.
- `styling-guidelines.md` owns animated styles, gestures, worklets, motion tokens, shared transitions, and cleanup rules.
- Keep `className` stable and pass dynamic transforms, opacity, and interpolated values through Reanimated `style`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jp2507-max) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
