## ui-library

> This is a React UI component library with MUI-style theming.

# Cursor Rules for UI Library

[include]
src/**/*.tsx
src/**/*.ts
src/**/*.tsx
.storybook/**/*.tsx
.storybook/**/*.ts

[exclude]
node_modules/**
dist/**
**/*.test.ts
**/*.test.tsx
**/*.stories.tsx

# Library Context
This is a React UI component library with MUI-style theming.
All components require ThemeProvider to be set up with a palette.
Components use theme colors via the useTheme() hook with inline styles.

# Component Structure
- Components are in src/components/
- Theme system is in src/theme/
- All exports are in src/index.ts
- Components use Tailwind CSS for styling
- TypeScript types are fully exported

# Theming
- ThemeProvider is REQUIRED
- Use createTheme() to define palettes (MUI-style)
- All components use useTheme() hook to access theme colors directly
- Colors are applied via inline styles, not CSS variables
- Supports light/dark mode via useThemeMode()

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dylanGerlach) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
