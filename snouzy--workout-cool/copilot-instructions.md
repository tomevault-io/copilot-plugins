## workout-cool

> This project follows **Next.js (App Router)** and is structured using **Feature-Sliced Design (FSD)** for modularity, scalability, and clear

## 🧑‍💻 Development Guidelines

This project follows **Next.js (App Router)** and is structured using **Feature-Sliced Design (FSD)** for modularity, scalability, and clear
separation of concerns.

Use this prompt and coding standards to ensure consistency across the codebase:

---

### 🔧 Code Style and Structure

- Write concise, expressive, and idiomatic **TypeScript**
- Use **functional programming** patterns (avoid classes and side effects)
- Prefer **composition** over inheritance, and modularization over duplication
- Organize each `feature/`, `entity/`, or `widget/` with:

  - model/ → logic (React Query, actions, hooks)
  - schema/ → Zod schemas for validation ui/ → client components (TSX)
  - lib/ → pure helper functions
  - types/ → interfaces & TS types

- All external dependencies (**API**, `localStorage`, `Date`) must be **abstracted** in `shared/lib/`
- Avoid direct calls to:
- `fetch` → use actions or `shared/api/`
- `new Date()` → use `shared/lib/date` abstraction
- `localStorage` → wrap in `shared/lib/storage`

---

### 🧠 Naming Conventions

- Use `kebab-case` for **directories** (e.g. `features/auth/signup`)
- Use **named exports** (no default exports for components)
- Use descriptive names with **auxiliary verbs** (e.g. `isLoading`, `hasError`, `canSubmit`)
- Components:
- Pure UI: `src/components/ui/`
- Shared logic: `src/shared/lib/`
- Composition: `src/widgets/`

---

### 📐 TypeScript Usage

- Use `interface` over `type` for objects
- Avoid `enum`; use `as const` object maps instead
- Use `infer` and `z.infer<typeof schema>` for accurate form types
- Types live in `types/` or colocated with usage

---

### 📦 Feature Architecture

**Keep React component logic inside the relevant feature:**

features/auth/signup/ ├── model/ → useSignUp.ts, signup.action.ts ├── schema/ → signup.schema.ts ├── ui/ → signup-form.tsx

If reusable between many features (e.g. `User`, `Link`, `Session`), move logic to `entities/`.

---

### 🧪 Error Handling & Validation

- Use **Zod** for schema validation
- Prefer early returns & guard clauses
- Use `ActionError` in server actions and handle them with `next-safe-action`
- Wrap React components in `ErrorBoundary` (or `shared/ui/ErrorBoundaries.tsx`)
- Display user-friendly errors via `toast()` or `<Alert />`

---

### 💅 UI & Styling

- Use **Shadcn UI**, **Radix**, and **Tailwind CSS** with **mobile-first** responsive design
- Design theme:

  - **Minimal**, professional with a **slightly playful touch**
  - Inspired by **Apple**, tailored to fitness coaches
  - Emphasize visuals: badges, progress bars, illustrations
  - Use `lucide-react` icons, subtle borders, hover feedback
  - Avoid drop shadows; prefer light borders and soft hover effects

- Animations:

  - Elegant and performant (use `framer-motion` if needed)
  - Use `transition`, `duration-xxx`, and `ease-xxx` from Tailwind

- UX Principles:

  - Clear hierarchy
  - Responsive: no overflow, no overlap
  - All buttons and interactive elements should provide feedback
  - Use @tailwind.config.ts for the theme.

- **UI Stack**:

  - **Shadcn UI**, **Radix UI**, and **Tailwind CSS** (mobile-first approach)
  - Icons: **lucide-react**

- **Design Language**:

  - 🎨 **Modern & minimalist**, inspired by **Apple’s design system**, with a **slightly more colorful palette**
  - Interface should be **clean**, **cohesive**, and **functional** without sacrificing features
  - Avoid drop shadows; prefer **subtle borders** where relevant
  - Ensure a **clear visual hierarchy** and **intuitive navigation**

- **Interactive Components**:

  - Buttons and inputs must be **elegant**, with **subtle visual feedback** (hover, click, validation)
  - Use **addictive micro-interactions** sparingly to enhance engagement without clutter

- **Animations**:

  - Use Tailwind’s built-in utilities: `transition`, `duration-xxx`, `ease-xxx` for basic transitions
  - Use `framer-motion` for advanced animations only if necessary
  - ✅ **Performance comes first**: animations must be smooth and lightweight

- **Responsiveness**:

  - Fully responsive layout: **no overlapping**, **no overflow**
  - Consistent behavior across all devices, from mobile to desktop

- **User Experience**:
  - All interactive elements must provide **clear visual feedback**
  - Interfaces should remain **simple to navigate**, even when **feature-rich**

---

### 🧱 Rendering & Performance

- Favor **Server Components** (`RSC`) and SSR for pages and logic
- Limit `'use client'` usage — only where needed:
  - form states, event listeners, animations
- Wrap all client components in `<Suspense />` with fallback
- Use dynamic import for non-critical UI (e.g. `Dialog`, `Chart`)
- Optimize media:
  - Use **WebP** images with width/height
  - Enable lazy loading where possible

---

### 🔍 Data, Forms, Actions

- Use `@tanstack/react-query` for client state
- Use `next-safe-action` for server mutations and queries
- All actions should:
  - Have clear schema (`schema/`)
  - Model expected errors with `ActionError`
  - Return typed output
  - Use the clientAction from `@/shared/api/safe-actions`
- Use `Form`, `FormField`, `FormMessage` from Shadcn for all forms

---

### 🧭 Routing & Navigation

- All routes defined in `app/`, avoid logic here
- Use constants in `shared/constants/paths.ts`
- For search parameters, use `nuqs` (`useQueryState`) — never manipulate `router.query` directly
- Follow Next.js App Router standards for layouts and segments

---

- [Feature-Sliced Design](https://feature-sliced.design/)
- [Shadcn UI](https://ui.shadcn.com/)
- [Zod](https://zod.dev/)

---
> Source: [Snouzy/workout-cool](https://github.com/Snouzy/workout-cool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
