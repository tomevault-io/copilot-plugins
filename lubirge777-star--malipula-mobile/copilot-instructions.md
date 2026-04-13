## malipula-mobile

> This is an Expo React Native e-commerce mobile application for "Malipula Suits" (custom suit tailoring). The app uses expo-router for file-based routing, NativeWind (Tailwind CSS) for styling, Zustand for state management, and Supabase for backend services.

# AGENTS.md - Malipula Mobile Development Guide

## Overview

This is an Expo React Native e-commerce mobile application for "Malipula Suits" (custom suit tailoring). The app uses expo-router for file-based routing, NativeWind (Tailwind CSS) for styling, Zustand for state management, and Supabase for backend services.

## Tech Stack

- **Framework**: Expo SDK 54 with React Native 0.81.5
- **Navigation**: expo-router (file-based routing in `app/` directory)
- **Styling**: NativeWind v4 (Tailwind CSS for React Native)
- **State Management**: Zustand
- **Forms**: React Hook Form + Zod validation
- **Backend**: Supabase (auth, database, storage)
- **TypeScript**: Strict mode enabled (tsconfig.json)

---

## Build & Development Commands

### Core Commands
```bash
# Start development server
npm start                # or: expo start
npm run android          # Start with Android emulator
npm run ios              # Start with iOS simulator
npm run web              # Start web build
```

### Type Checking
```bash
# Run TypeScript compiler to check types (no emit)
npx tsc --noEmit

# TypeScript with project references
npx tsc -p tsconfig.json --noEmit
```

### Linting
```bash
# Note: No ESLint config exists yet - consider adding:
# npm install eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

### Testing
```bash
# Note: No Jest config exists yet. To add testing:
# npm install --save-dev jest @testing-library/react-native jest-expo
# npx jest --watch
```

---

## Code Style Guidelines

### File Organization
- **Routes**: `app/` directory - expo-router file-based routing
  - Use `(group)` syntax for route groups (e.g., `(tabs)`, `(auth)`)
  - Dynamic routes: `[param].tsx`
- **Components**: `src/components/ui/` - reusable UI components
- **Stores**: `src/stores/` - Zustand state stores
- **Theme**: `src/theme/` - colors, fonts, spacing, typography
- **Lib**: `src/lib/supabase/` - client, queries, auth utilities
- **Types**: `src/types/index.ts` - shared TypeScript interfaces

### Naming Conventions
- **Components**: PascalCase (e.g., `Button.tsx`, `ProductCard.tsx`)
- **Hooks**: camelCase with `use` prefix (e.g., `useAuth()`, `useCart()`)
- **Stores**: camelCase with `Store` suffix (e.g., `useAuthStore`)
- **Types/Interfaces**: PascalCase (e.g., `ButtonProps`, `User`)
- **Files**: kebab-case (e.g., `auth-store.ts`, `bottom-sheet.tsx`)
- **Constants**: SCREAMING_SNAKE_CASE for configuration values

### TypeScript Guidelines
- Use `strict: true` in tsconfig.json
- Prefer explicit type annotations for function parameters
- Use `interface` for object shapes, `type` for unions/aliases
- Import types with `import type { User }` syntax
- Avoid `any` - use `unknown` when type is truly unknown

### Import Order (recommended)
1. React/Next imports (from 'react', 'expo-router')
2. Third-party library imports
3. Internal absolute imports (`@/components/...`)
4. Relative imports (`.`, `..`)
5. Type imports (separated with `import type`)

### Component Patterns
- Use named exports for all components (not default)
- Destructure props with defaults in function signature
- Keep components focused (single responsibility)
- Extract complex logic to custom hooks

### State Management (Zustand)
- Create stores in `src/stores/` with descriptive names
- Use TypeScript interfaces for state shape
- Export typed hooks (e.g., `export const useAuthStore = create<AuthState>(...)`)
- Avoid combining unrelated state in single store

### Styling (NativeWind/Tailwind)
- Use Tailwind classes in JSX for most styling
- Use theme tokens from `src/theme/` for colors/spacing when needed
- Avoid inline StyleSheet for complex dynamic styles
- Use `className` prop (NativeWind)

### Error Handling
- Use try/catch for async operations
- Display user-friendly error messages via Toast/Alert
- Log errors for debugging (use console.error with context)
- Handle Supabase errors gracefully with fallback UI

### Theme Usage
- Colors: `src/theme/colors.ts` - brand colors (gold, navy, ivory, charcoal)
- Typography: `src/theme/fonts.ts` - custom fonts (Inter, Playfair Display)
- Spacing: Use NativeWind spacing (`p-4`, `m-2`) or theme values
- BorderRadius: Use nativewind classes (`rounded-lg`, `rounded-xl`)

---

## Project Structure Reference

```
app/                      # Expo Router pages
├── (auth)/               # Auth routes (sign-in, sign-up)
├── (tabs)/               # Tab navigation (home, shop, cart, bookings, account)
├── account/              # Account subpages
├── shop/                 # Shop/product pages
├── checkout/             # Checkout flow
└── booking/              # Booking flow

src/
├── components/ui/       # Reusable UI components
├── lib/supabase/         # Supabase client & queries
├── stores/               # Zustand stores
├── theme/                # Theme configuration
└── types/                # TypeScript types
```

---

## Adding New Features

1. **New Screen**: Add route file in `app/` directory following expo-router conventions
2. **New Component**: Create in `src/components/ui/` with exports in `index.ts`
3. **New Store**: Create in `src/stores/` using Zustand pattern
4. **New Type**: Add to `src/types/index.ts`
5. **New Query**: Add to `src/lib/supabase/queries.ts`

---

## Common Issues & Solutions

- **Type errors**: Run `npx tsc --noEmit` to check
- **NativeWind not applying**: Ensure `babel-preset-expo` has `{ jsxImportSource: "nativewind" }`
- **Supabase connection**: Check `.env` matches `.env.example` structure
- **Missing fonts**: Ensure fonts are loaded in app entry (see `src/theme/fonts.ts`)

---

## Environment Variables

Create `.env` from `.env.example`:
```
EXPO_PUBLIC_SUPABASE_URL=your_supabase_url
EXPO_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lubirge777-star) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
