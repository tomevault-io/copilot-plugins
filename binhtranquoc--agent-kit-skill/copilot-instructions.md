## agent-kit-skill

> > **Goal:** Build performant, accessible, and maintainable UI applications.

# Frontend General Rules

> **Goal:** Build performant, accessible, and maintainable UI applications.

---

## 1. Design System & Colors

### Color Variables (CSS Custom Properties)
```css
:root {
  /* Primary palette */
  --color-primary: #3b82f6;
  --color-primary-hover: #2563eb;
  --color-secondary: #6b7280;
  
  /* Semantic colors */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-error: #ef4444;
  
  /* Neutral */
  --color-background: #ffffff;
  --color-surface: #f9fafb;
  --color-text: #111827;
  --color-text-muted: #6b7280;
}
```

**Rules:**
- ❌ **Never hardcode colors** - Use CSS variables or design tokens
- ✅ Create dark mode using CSS variables swap
- ✅ Use semantic color names (not `--blue-500`)

---

## 2. UI/UX Principles

### Loading States
- ✅ Use **Skeleton loading** over spinners for content
- ✅ Use **Optimistic updates** for user actions
- ✅ Show inline loading states for buttons

### Error Handling
- ✅ Use **Error Boundaries** for unexpected errors
- ✅ Show user-friendly error messages with retry action
- ❌ Never show raw error stack to users

### Responsive Design
- ✅ **Mobile-first** approach
- ✅ Use `rem`/`em` for spacing, not `px`
- ✅ Max container width: `1280px`
- ✅ Breakpoints: `sm: 640px`, `md: 768px`, `lg: 1024px`, `xl: 1280px`

---

## 3. State Management

### When to Use What
| State Type | Solution |
|-----------|----------|
| **Local UI state** | `useState`, component state |
| **Server state** | React Query, SWR, TanStack Query |
| **Global UI state** | Context API, Zustand (keep minimal!) |
| **Form state** | React Hook Form, Formik |

**Rules:**
- ❌ **Never use Redux for everything** - It's overkill for most apps
- ❌ **Avoid deep global state** - Prefer server state caching
- ✅ Keep global state to: auth, theme, i18n only

---

## 4. Performance Optimization

### Must-Have Optimizations
1. **Code Splitting** - Lazy load routes and heavy components
2. **Image Optimization** - WebP/AVIF format, lazy loading
3. **Virtualization** - Use virtual scrolling for 50+ items list
4. **Memoization** - `useMemo`, `useCallback` for expensive operations
5. **Debounce** - Search inputs, window resize handlers

### Example: Lazy Loading
```tsx
// Route-based code splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));

// Component lazy loading
const HeavyChart = lazy(() => import('./components/HeavyChart'));
```

---

## 5. Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| **Components** | PascalCase | `UserProfile.tsx` |
| **Hooks** | camelCase + `use` prefix | `useAuth.ts` |
| **Utils** | camelCase | `formatDate.ts` |
| **Constants** | SCREAMING_SNAKE | `API_ENDPOINTS.ts` |
| **CSS Modules** | kebab-case | `user-profile.module.css` |

---

## 6. Component Structure

### File Organization (Feature-based)
```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── LoginForm.test.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   ├── api/
│   │   │   └── authApi.ts
│   │   └── index.ts
│   └── dashboard/
├── shared/
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   └── Button.module.css
│   │   └── Input/
│   └── hooks/
└── utils/
```

### Component Best Practices
- ✅ **Single Responsibility** - One component, one purpose
- ✅ **Props destructuring** at function signature
- ✅ **Named exports** over default exports
- ❌ **No prop drilling** beyond 2 levels - Use Context or composition

```tsx
// Good: Props destructured at signature
const Card = ({ title, children, className }: CardProps) => {
  return (
    <div className={cn('card', className)}>
      <h2>{title}</h2>
      {children}
    </div>
  );
};
```

---

## 7. Accessibility (a11y)

### Minimum Requirements
- ✅ All images have `alt` text
- ✅ All interactive elements are keyboard accessible
- ✅ Color contrast ratio ≥ 4.5:1 for text
- ✅ Focus indicators visible
- ✅ ARIA labels for icon-only buttons
- ❌ **Never remove focus outline** without replacement

---

## 8. Icons & Assets

### Rules
- ❌ **Limit icon usage** - Only essential icons (nav, actions)
- ❌ **No decorative stickers** - Keep UI professional
- ✅ Use **SVG sprite** or icon library (Lucide, Heroicons)
- ✅ Lazy load images below the fold

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/binhtranquoc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
