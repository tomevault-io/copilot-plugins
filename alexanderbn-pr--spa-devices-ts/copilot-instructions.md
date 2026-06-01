## spa-devices-ts

> **Last updated**: Mayo 2026

# AGENTS.md: React Senior Developer Guide

**Last updated**: Mayo 2026  
**Tech Stack**: React 19 + TypeScript + Vite + TanStack Query + Zustand + Vitest

---

## SECCIÓN 1: Fundamentos del Proyecto

### 1.1 Tech Stack
- React 19 + TypeScript + Vite
- TanStack Query (server state)
- Zustand (client state)
- React Router v6
- SCSS + CSS Modules
- Vitest + React Testing Library

### 1.2 Folder Structure

**Standard React (without features):**
```
src/
├── components/        # Reusable UI (Button, Input, Layout)
├── pages/             # Route pages (containers)
├── hooks/             # Global hooks (useAuth, useDebounce)
├── services/          # API calls per domain (menus.api.ts, recipes.api.ts)
├── store/             # Zustand stores
├── types/             # Global TypeScript types
├── utils/             # Pure helpers
├── constants/         # Global constants
├── styles/            # Global SCSS, themes, variables
├── assets/            # Images, fonts, icons
├── config/            # Env vars, API client setup
├── routes/            # Route definitions
├── App.tsx
└── main.tsx
```

**Feature-based structure:**
```
/features/users/
  - components/
  - hooks/
  - pages/
  - types/
  - services/
```

### 1.3 Architectural Rules
- ✅ One component per file
- ✅ DO NOT fetch data inside components
- ✅ DO NOT mix business logic with UI
- ✅ DO NOT duplicate logic (extract hooks)
- ✅ Group related files (component + styles + test)

---

## SECCIÓN 2: Código Limpio y Convenciones

### 2.1 Code Style
- Arrow functions only
- Prefer `const` over `let`
- Early returns (exit early, avoid nesting)
- Avoid nested conditionals
- Keep components small (< 150 lines)

### 2.2 TypeScript Rules
- Always type props explicitly
- Avoid `any` (use `unknown` if needed)
- Use interfaces for props
- Type all API responses
- Use union types when applicable

### 2.3 Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Components | PascalCase | `UserCard.tsx` |
| Hooks | camelCase, `use` prefix | `useAuth`, `useFetchData` |
| API files | `entity.api.ts` | `menus.api.ts` |
| Types | `entity.types.ts` | `menu.types.ts` |
| Query keys | `['entity']` | `['menus']` |
| Functions | camelCase, verb + intent | `getUserData()`, `handleRemove()` |
| Variables | camelCase | `filterProducts` |
| Booleans | `is`/`has`/`can` prefix | `isLoading`, `hasPermission` |
| Interfaces/Types | PascalCase | `UserProps`, `ApiResponse` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRIES`, `API_BASE_URL` |
| CSS Files | Same name as component | `LoginForm.scss` |

### 2.4 ESLint Compliance

**MANDATORY before delivery:**
- Resolve ALL errors/warnings
- Remove unused imports/variables/functions
- Use `eslint --fix` then review manually
- No `console.log` in production (only `console.error` for errors)

---

## SECCIÓN 3: React Core & Hooks

### 3.1 Data Fetching

- **NEVER** use `fetch` inside `useEffect`
- Use **TanStack Query**: `useQuery` (GET), `useMutation` (mutations)
- Encapsulate in custom hooks: `useMenus()`, `useUpdateMenu()`

### 3.2 Loading States & UX
- Prefer **skeletons** over spinners
- Avoid layout shifts
- Wrap async components with **Suspense**
- Keep UI responsive during loading

### 3.3 Performance Rules

- `React.memo` es para componentes que reciben props complejas pero **rara vez cambian**; **no es gratis**, compara shallowly
- `useMemo` solo cuando el cálculo es genuinamente pesado (O(n²) o mayor, o creación de objetos/arrays que disparan effects)
- `useCallback` solo cuando la función se pasa a un componente memoizado o a un effect con dependencias
- **NO** envuelvas todo en memo "por si acaso"; el costo de la comparación puede superar el del re-render
- Virtualiza listas largas (>100 items visibles): `react-window` o `react-virtuoso`
- Evita crear objetos/funciones inline en el render si se pasan a componentes memoizados (rompen la memoización)

### 3.4 Error Handling
- Use **Error Boundaries** for UI crashes
- Never swallow errors silently
- Always propagate errors up

### 3.5 React 19 / Next.js App Router

- Server Components por defecto; Client Components solo cuando necesites interactivity (hooks, browser APIs)
- **NUNCA** uses `useEffect` para fetching inicial en Server Components (fetch directamente en el componente)
- Server Actions para mutaciones; evita APIs REST intermedias si no son necesarias
- `useActionState` para manejar estado de formularios server-side
- Marca explícitamente `"use client"` solo en la frontera mínima necesaria

### 3.6 Reglas Fundamentales de Hooks

**Leyes de los Hooks (inquebrantables):**
- Solo llamar Hooks en el nivel superior (nunca en loops, conditions, o funciones anidadas)
- Solo llamar Hooks desde componentes React o Custom Hooks (nunca desde funciones regulares)
- El orden de llamada de los Hooks debe ser idéntico en cada render

### 3.7 useEffect: Reglas de Oro

- **NO** uses `useEffect` para sincronizar estado derivado (usa `useMemo` o calcula en render)
- **NO** uses `useEffect` para manejar eventos del usuario (usa handlers directamente)
- **NO** uses `useEffect` para fetching si tienes TanStack Query/SWR
- Si necesitas `useEffect`, siempre incluye el array de dependencias completo y correcto
- Si el linter de dependencias te advierte, escúchalo o documenta por qué lo ignoras con `eslint-disable-next-line` + comentario explicativo

### 3.8 Side Effects Disciplina

- Toda lógica impura (localStorage, WebSockets, timers, subscriptions) va en `useEffect` o en event handlers
- Limpia siempre los effects: `removeEventListener`, `clearInterval`, `unsubscribe`, `abortController.abort()`
- Usa **AbortController** para fetch nativo cuando no uses TanStack Query
- **NUNCA** llames a setState sincrónicamente dentro de un effect sin condición de guarda (puede causar loops infinitos)

### 3.9 Refs vs State

- Usa `useRef` para valores que no deben disparar re-render (interval IDs, elementos DOM, flags de montaje)
- **NUNCA** leas/escribas `ref.current` durante el render (solo en effects o handlers)

---

## SECCIÓN 4: Gestión de Estado

### 4.1 Estado Global vs Local

- **NO** uses Zustand/Redux para estado que puede vivir en un componente padre
- Levanta el estado lo mínimo indispensable (**State Colocation**)
- Si dos componentes no relacionados necesitan el mismo dato → estado global
- **Regla**: Local primero, global solo si necesario

### 4.2 Server State vs Client State

- **Server State** (datos del backend): TanStack Query
- **Client State** (UI, filters, etc): Zustand o estado local
- **NUNCA** dupliques datos entre server state y client state

### 4.3 Inmutabilidad

- NUNCA mutes arrays/objetos directamente en el estado
- Usa **spread operator**, `map`, `filter`, o librerías como Immer si la lógica es compleja
- React depende de referencias para re-renderizar; la mutación silenciosa = bugs de sincronización

---

## SECCIÓN 5: Patrones de Componentes

### 5.1 Composición sobre Configuración

- Prefiere componer componentes con `children` y slots en lugar de props booleanas gigantes
- ✅ Bueno: `<Modal><ModalHeader /><ModalBody /></Modal>`
- ❌ Malo: `<Modal showHeader showBody />`

### 5.2 Container/Presentational Pattern

- **Containers** (smart): Conectan a datos, manejan lógica, usan hooks
- **Presentational** (dumb): Solo pintan UI, reciben todo por props
- Los presentational NO saben de APIs ni stores

### 5.3 Controlled vs Uncontrolled Inputs

- Usa **controlled** cuando necesitas validación en tiempo real, formularios complejos, o sincronización con estado global
- Usa **uncontrolled** + `ref` para formularios simples, inputs de búsqueda con debounce
- **NUNCA** mezcles ambos patrones en el mismo input (ej: tener `value` y `defaultValue` simultáneamente)

### 5.4 Render Props vs HOCs vs Custom Hooks

- **Prioridad**: Custom Hooks > Render Props > HOCs
- Los HOCs están obsoletos para lógica compartida; úsalos solo para cross-cutting concerns (ej: analytics, permisos)
- Los Render Props son válidos para casos de inversión de control complejos, pero prioriza hooks

### 5.5 Custom Hooks: Best Practices

- Extrae lógica reutilizable en hooks personalizados
- Hooks deben ser composables (pueden usar otros hooks)
- Documenta dependencias externas del hook
- Testea hooks con `renderHook` de React Testing Library

---

## SECCIÓN 6: API Layer & Data Integration

### 6.1 File Organization

- One file per domain: `menus.api.ts`, `recipes.api.ts`, `devices.api.ts`
- All CRUD operations in the same file
- Raw API types private to the API file
- Mappers mandatory: `mapMenuFromApi`, `mapMenuToApi`

### 6.2 Example: `menus.api.ts`

```typescript
import { supabase } from '@/config/supabaseClient';
import { Menu, MenuUpdateDTO } from '../types/menu.types';
import { mapMenuFromApi } from './menus.mappers';

export const menusApi = {
  getAll: async () => {
    const { data, error } = await supabase
      .from('menus')
      .select('*, day:days(id, name), moment:moments(id, name)')
      .returns<ApiMenuRow[]>();

    if (error) throw new Error(error.message);
    return (data ?? []).map(mapMenuFromApi);
  },

  update: async (id: string, updates: Partial<MenuUpdateDTO>) => {
    const { error } = await supabase
      .from('menus')
      .update(updates)
      .eq('id', id);

    if (error) throw new Error(error.message);
  },

  clearAllRecipes: async () => {
    const { error } = await supabase
      .from('menus')
      .update({ recipe_id: null })
      .not('recipe_id', 'is', null);

    if (error) throw new Error(error.message);
  },
} as const;

// Raw API types (never use outside this file)
interface ApiMenuRow {
  id: string;
  menu_name: string;
  created_at: string;
  // ... other raw fields
}
```

### 6.3 Supabase Specific Rules

- Always handle `error` from Supabase responses
- Use `.returns<T>()` for type safety
- For relations: `day:days(id, name)` not `day:days(*)`
- Bulk updates: use `.in()` or `.not()` filters, never loop individual updates
- `recipe_id: null` to clear a relation (DO NOT delete rows)

---

## SECCIÓN 7: UI, Accesibilidad y Diseño

### 7.1 Design System

See `DESIGN.md` for full reference.

| Element | Value |
|---------|-------|
| Primary CTA | `#0071e3` |
| Page bg | `#f5f5f7` |
| Text | `#1d1d1f` |
| Card shadow | `rgba(0,0,0,0.22) 3px 5px 30px 0px` |

**Typography:**
- Display: SF Pro Display (≥20px)
- Body: SF Pro Text (<20px)
- Negative letter-spacing at all sizes

### 7.2 CSS/SCSS Guidelines

- **Maximum 3 nesting levels** in SCSS
- Follow **BEM methodology**:
  - **Block**: `.login-form`
  - **Element**: `.login-form__input`
  - **Modifier**: `.login-form__input--error`
- Use design tokens and variables
- Prefer semantic class names

### 7.3 Semantic HTML & Accessibility Rules

### PROHIBITED: `<div>` for semantic structure

**❌ NEVER use `<div>` as a generic container for semantic sections.**

```tsx
// ❌ WRONG - div for everything
<div className="page">
  <div className="header">...</div>
  <div className="main">
    <div className="article">...</div>
    <div className="sidebar">...</div>
  </div>
  <div className="footer">...</div>
</div>
```

**✅ ALWAYS use semantic HTML5 elements:**

```tsx
// ✅ CORRECT - semantic structure
<main className="page">
  <header>...</header>
  <section>
    <article>...</article>
    <aside>...</aside>
  </section>
  <footer>...</footer>
</main>
```

### `<div>` Usage Rules

| Scenario | Element to Use | Exception |
|----------|---------------|-----------|
| Page wrapper | `<main>` | Only if it's the unique main content |
| Navigation links | `<nav>` | — |
| Header/banner | `<header>` | — |
| Footer | `<footer>` | — |
| Self-contained content | `<article>` | Blog posts, cards, comments |
| Thematic grouping | `<section>` | With heading (`<h2>`–`<h6>`) |
| Sidebar/tangential content | `<aside>` | Related links, ads, quotes |
| Details disclosure | `<details>` | No JS needed for toggle |
| Dialog/modal | `<dialog>` | Use `showModal()` / `close()` |
| Figure with caption | `<figure>` + `<figcaption>` | Images, diagrams, code blocks |
| List of items | `<ul>` / `<ol>` / `<dl>` | Never `<div>` for lists |
| Tabular data | `<table>` | Never `<div>` grids for data |
| Form sections | `<fieldset>` + `<legend>` | Group related controls |
| Time data | `<time datetime="...">` | Machine-readable dates |
| Marked/highlighted text | `<mark>` | Search results, highlights |
| Progress indicator | `<progress>` | With `value` and `max` |
| Meter/gauge | `<meter>` | With `min`, `max`, `low`, `high` |

### When `<div>` IS Allowed

Use `<div>` **ONLY** for:
1. **Pure styling wrappers** — CSS layout helpers, flex/grid containers with no semantic meaning
2. **Animation containers** — motion/transition wrappers without content meaning
3. **Third-party component roots** — when a library requires a neutral container

```tsx
// ✅ ALLOWED - pure styling, no semantic intent
<div className="flex-wrapper">
  <section>...</section>
  <section>...</section>
</div>

// ✅ ALLOWED - animation container
<div className="fade-in-wrapper">
  <article>...</article>
</div>
```

### Accessibility Checklist

- [ ] Single `<h1>` per page (main topic)
- [ ] Heading hierarchy: no skipped levels (`h1` → `h2` → `h3`)
- [ ] All interactive elements keyboard-accessible
- [ ] `aria-label` or `aria-labelledby` on `<nav>` with multiple instances
- [ ] `aria-expanded` on `<details>` if controlling via JS
- [ ] Focus management on `<dialog>` open/close
- [ ] Color contrast ≥ 4.5:1 for normal text
- [ ] Touch targets ≥ 48×48px

---

---

## SECCIÓN 9: Testing & Quality Assurance

### 9.1 Testing Rules

- **Test behavior, not implementation**: "el usuario ve el menú" > "el componente llama a useMenus"
- Use **semantic queries**: `getByRole`, `getByLabelText` > `getByTestId`
- Mock services/APIs, **NEVER** mock React internals or component implementations
- Each test must be independent (no shared state between `it()` blocks)
- Use `userEvent` instead of `fireEvent` for realistic interactions
- Test error and loading states, not just the happy path

### 9.2 Testing Setup

- Framework: **Vitest** + React Testing Library
- Config: See `vitest.config.ts`
- Coverage target: ≥ 80% for critical paths
- Focus on integration tests over unit tests

---

## SECCIÓN 10: Available Skills & Tools

### 10.1 Available Skills

| Skill | Trigger | Focus |
|-------|---------|-------|
| **frontend-design** | Create distinctive, production-grade frontend interfaces | UI design, styling, aesthetics |
| **react-19** | React 19 patterns with React Compiler | React 19 features, hooks |
| **react-avanced** | Performance optimization from Vercel Engineering | Performance, optimization, rendering |
| **react-form** | React forms with Zod + React Hook Form | Forms, validation, handling |
| **react-query** | Server state management with TanStack Query | Data fetching, caching, state |
| **react-seo-structure** | SEO optimization and structured data | SEO, metadata, Core Web Vitals |
| **react-table** | Production-grade tables with TypeScript | Tables, sorting, filtering |
| **scss** | SCSS styling best practices | Styling, CSS architecture, design tokens |
| **typescript-advanced-types** | Advanced TypeScript type system | Type safety, generics, utilities |
| **vitest** | Fast unit testing with Vitest | Testing, coverage, mocking |

### 10.2 Skill Triggers Reference

| Context | Skill(s) | Key Rule |
|---------|----------|----------|
| **Architecture & Structure** | react-form, typescript-advanced-types | Separate UI/hooks/services, one component per file |
| **Code Style & Performance** | react-avanced | Keep components small, avoid nested conditionals |
| **TypeScript & Types** | typescript-advanced-types | Always type props explicitly, avoid `any` |
| **Forms & Validation** | react-form | Use React Hook Form + Zod, separate logic from UI |
| **Data Fetching** | react-query | Use TanStack Query (useQuery, useMutation) |
| **Performance** | react-avanced | Memoize when needed, avoid unnecessary global state |
| **Tables & Grids** | react-table | Generic table pattern with TypeScript |
| **Testing** | vitest | Test behavior, not implementation |
| **Styling** | scss, frontend-design | BEM methodology, design tokens, Apple system |
| **SEO & Metadata** | react-seo-structure | JSON-LD structured data, optimize meta tags |
| **React 19** | react-19 | Use React 19 patterns, React Compiler compatible |

### 10.3 Commands

| Command | Script | Purpose |
|---------|--------|---------|
| Dev | `npm run dev` | Start development server |
| Build | `npm run build` | Production build |
| Lint | `npm run lint` | ESLint check |
| Test | `npm run test` | Run tests in watch mode |
| Test (run) | `npm run test:run` | Run tests once |
| Test (coverage) | `npm run test:coverage` | Generate coverage report |

---

## SECCIÓN 11: Quick Reference Checklist

### Pre-Delivery Checklist

- [ ] ✅ All ESLint errors/warnings resolved
- [ ] ✅ No unused imports, variables, or functions
- [ ] ✅ No `console.log` in production code (only `console.error`)
- [ ] ✅ All components properly typed (no `any`)
- [ ] ✅ Semantic HTML used (no unnecessary `<div>` wrappers)
- [ ] ✅ Accessibility checklist passed (contrast, focus, keyboard)
- [ ] ✅ Tests written for critical paths (>80% coverage)
- [ ] ✅ Error boundaries in place
- [ ] ✅ Loading/skeleton states implemented
- [ ] ✅ API layer properly organized (one file per domain)
- [ ] ✅ Mappers defined for API responses
- [ ] ✅ No data fetching in component bodies
- [ ] ✅ Barrel exports created for reusable modules
- [ ] ✅ BEM naming for CSS classes
- [ ] ✅ Comments added for complex logic
- [ ] ✅ No circular dependencies in imports

---
> Source: [alexanderbn-pr/spa-devices-ts](https://github.com/alexanderbn-pr/spa-devices-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
