## askify

> Askify là một Topic-Based Q&A Platform được xây dựng với React + Vite, TailwindCSS, styled-components và Acrylic Glass Effect UI. Dự án sử dụng theme-aware design system hỗ trợ cả light và dark theme.

# Askify Project Structure & Guidelines

## Project Overview

Askify là một Topic-Based Q&A Platform được xây dựng với React + Vite, TailwindCSS, styled-components và Acrylic Glass Effect UI. Dự án sử dụng theme-aware design system hỗ trợ cả light và dark theme.

## Key Architecture Files

### Theme System
- [frontend/src/styles/theme.ts](mdc:frontend/src/styles/theme.ts) - Centralized theme system với colors, mixins và theme-aware utilities
- [frontend/src/styles/sharedStyles.ts](mdc:frontend/src/styles/sharedStyles.ts) - Shared acrylic effects và styling mixins  
- [frontend/src/styles/acrylic.css](mdc:frontend/src/styles/acrylic.css) - CSS cho acrylic glass effects
- [frontend/src/hooks/useTheme.ts](mdc:frontend/src/hooks/useTheme.ts) - Theme management hook

### UI Components System
- [frontend/src/components/ui/](mdc:frontend/src/components/ui) - Reusable theme-aware UI components
- [frontend/src/components/ui/index.ts](mdc:frontend/src/components/ui/index.ts) - Centralized exports
- [frontend/src/components/ui/README.md](mdc:frontend/src/components/ui/README.md) - Comprehensive usage guide

### Shared Components  
- [frontend/src/components/shared/](mdc:frontend/src/components/shared) - Acrylic components (Button, Input, Container)
- [frontend/src/components/shared/AcrylicProvider.tsx](mdc:frontend/src/components/shared/AcrylicProvider.tsx) - Theme provider
- [frontend/src/components/shared/BackgroundContainer.tsx](mdc:frontend/src/components/shared/BackgroundContainer.tsx) - Animated gradient backgrounds

### Layout System
- [frontend/src/layouts/MainLayout.tsx](mdc:frontend/src/layouts/MainLayout.tsx) - Main layout với navigation
- [frontend/src/layouts/BlankLayout.tsx](mdc:frontend/src/layouts/BlankLayout.tsx) - Clean layout cho auth pages

## Development Guidelines

### ✅ Best Practices

1. **Always use UI components** thay vì Tailwind classes trực tiếp cho acrylic effects
2. **Use theme-aware mixins** từ `styles/theme.ts` cho consistent styling
3. **Create styled components** thay vì inline Tailwind cho custom styling
4. **Test both light and dark themes** khi phát triển components mới
5. **Follow Vietnamese naming** trong comments và documentation

### Component Creation Pattern

```tsx
import styled from 'styled-components';
import { acrylicBase, themeAwareText } from '../../styles/theme';

const MyComponent = styled.div`
  ${acrylicBase}
  ${themeAwareText}
  // Additional styling
`;
```

### ❌ Avoid These Patterns

```tsx
// ❌ Don't use Tailwind for acrylic effects
<div className="bg-white bg-opacity-10 backdrop-blur-sm">

// ❌ Don't hardcode theme colors  
color: #ffffff;

// ❌ Don't create duplicate styling
```

## Responsive Design

- **Mobile-first approach** với breakpoints trong styled-components
- **Grid và Flexbox** cho layouts
- **Sticky navigation** với scroll detection
- **Touch-friendly** button sizes và spacing

## Theme System Usage

### Colors
```tsx
import { colors } from '../styles/theme';
// colors.light.text.primary, colors.dark.background.primary
```

### Mixins
```tsx
import { acrylicBase, themeAwareText, acrylicButton } from '../styles/theme';
```

## File Organization

```
frontend/src/
├── components/
│   ├── ui/           # Reusable UI components
│   ├── shared/       # Acrylic components
│   └── layout/       # Layout components
├── pages/           # Page components với folder structure
├── styles/          # Theme system và global styles
├── hooks/           # Custom hooks
└── layouts/         # Layout templates
```

## Performance Considerations

- **Lazy loading** cho components lớn
- **Memoization** cho expensive calculations
- **Optimized bundle** với Vite
- **CSS-in-JS optimization** với styled-components

## Commit Message Format

- `[UPDATE - DD/MM] - Description` cho updates
- `[FIX - DD/MM] - Description` cho bug fixes
- Always commit in English với Vietnamese comments trong code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/l4mbh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
