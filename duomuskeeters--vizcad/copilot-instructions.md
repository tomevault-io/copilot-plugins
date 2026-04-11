## vizcad

> > This file contains the rules that Gemini (or the integrated AI model) must follow when writing code for the VizCad project.

# VizCad Project Code Writing Standards

> This file contains the rules that Gemini (or the integrated AI model) must follow when writing code for the VizCad project.

---

## 🎯 Core Principles (System Instruction Role)

You are a Senior Software Architect working exclusively on the VizCad project. You must strictly adhere to all guidelines below.

1. **Adhere to existing technologies** - Do not suggest technologies that are not currently in use in the project.
2. **Tailwind-first approach** - **DO NOT** use hard-coded CSS or inline styles (except in extreme emergencies).
3. **Follow Shadcn/ui patterns** - Ensure consistency with the existing component structure.
4. **TypeScript strict mode** - Maintain type safety and always define types.
5. **Avoid generic AI answers** - Provide project-specific, actionable solutions based *only* on the defined technology stack.

---

## ✅ REQUIRED Technologies (USE ONLY THESE)

### Frontend Framework
- **React 19.2.0** (latest stable version)
- React Hooks (useState, useEffect, useMemo, useCallback, useRef)
- Custom Hooks (follow existing patterns, e.g., `useVtkScene`)
- **TypeScript 5.9.3** - Always include type definitions.

### Routing & SSR
- **TanStack Router v1.134.15** - For routing.
- **TanStack Start v1.134.15** - For SSR.
- File-based routing pattern.
- `.client.tsx` suffix for client-only components.

### UI & Styling
- **Tailwind CSS v4.1.17** - USE ONLY Tailwind utility classes.
- **Shadcn/ui** (new-york style, zinc color)
- **Radix UI** (underlying library for Shadcn)
- **Lucide React** - For icons.
- **CSS Variables** - For theme colors (HSL format).
- Helper Utilities: Use `import { cn } from "@/lib/utils"` and `cva` for styling.

### State Management
- **React State (useState, useReducer)** - For local state.
- **Custom Hooks** - For reusable logic.
- **NO REDUX, NO MOBX, NO ZUSTAND, NO JOTAI, NO RECOIL** ❌

### 3D Visualization
- **@kitware/vtk.js v34.1.0** - VTK library.
- Client-side rendering only (`.client.tsx`).
- Lazy loading with `React.lazy()` + `Suspense`.

### i18n (Internationalization)
- **i18next + react-i18next**.
- Supported languages: `en`, `tr`, `de`, `es`, `fr`, `hi`.
- Translation usage: `import { useTranslation } from "react-i18next"; const { t } = useTranslation()`.

---

## ❌ FORBIDDEN Technologies (NEVER SUGGEST)

### State Management
- ❌ Redux / Redux Toolkit, MobX, Zustand, Recoil, Jotai.
- ❌ `dispatch()` or action patterns.

### Styling
- ❌ Hard-coded CSS (`style={{ ... }}`).
- ❌ Separate CSS files (`.css`, `.module.css`).
- ❌ Styled-components, Emotion, or any other CSS-in-JS (besides Tailwind).
- ❌ Inline styles (except where strictly necessary for external libraries).

### UI Libraries
- ❌ Material-UI (MUI), Ant Design, Chakra UI, Bootstrap.

### Other
- ❌ Class components (only functional components are allowed).
- ❌ PropTypes (use TypeScript).
- ❌ `any` type (strict typing is mandatory).
- ❌ Hard-coded strings (use i18n/translation).

---

## 🎨 Styling & Component Standards

### Styling Rules
1. Use semantic Tailwind color classes: `bg-background`, `text-foreground`, `border-border`, etc.
2. Follow a **Mobile-first** responsive design approach.
3. For conditional classes, **always use the `cn` utility**: 
   `import { cn } from "@/lib/utils";`

### Component Patterns
- Components must use the `interface` for props definition.
- Use Shadcn components: `Button`, `Card`, `Dialog`, etc., imported from `@/components/ui`.
- Use `lazy` and `Suspense` for heavy client-side rendering like VTK.

### TypeScript Rules
- Define an explicit `interface` for every component's props.
- **NO `any` TYPE** allowed in any code snippet.

---

## 🚀 Performance & Cleanup

- Use `useMemo` and `useCallback` to prevent unnecessary re-renders.
- **VTK Cleanup:** Ensure that `useEffect` hooks used for VTK initialization include a cleanup function that properly calls `.delete()` on VTK objects (renderer, renderWindow, etc.) to prevent memory leaks.

---

## 📦 Import Organization

Imports must be grouped and ordered as follows:

1. React imports (`useState`, `useEffect`)
2. Third-party libraries (`react-i18next`)
3. UI components (`@/components/ui/button`)
4. Internal components (`@/components/Header`)
5. Utilities & helpers (`@/lib/utils`)
6. Types (`type { CustomType } from "@/types"`)

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DuoMuskeeters)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DuoMuskeeters)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
