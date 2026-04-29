## dashboard-web-template

> Use a skill **only when** the current feature, coding task, or explicit user/design mention requires it. Do not invoke skills by default.

# Skill Selection Guide (Chain of Thought)

Use a skill **only when** the current feature, coding task, or explicit user/design mention requires it. Do not invoke skills by default.

**Scope:** All skills are under `.cursor/skills/`. Paths are relative to the project root.

---

## When to use a skill (important)

Use a skill **only when one of the following is true**:

1. **The current case/task matches** — The work you are doing (e.g. writing a component, setting up TanStack Query, styling with Tailwind) clearly falls under the “Use when” of that skill. Do not load a skill “just in case”; only when the task actually needs it.

2. **The user explicitly mentions or requests it** — The user names a technology (e.g. “use Zustand”, “follow TanStack Query best practices”), asks to apply a pattern (e.g. “compound components”), or asks for a review (e.g. “check accessibility”). Then use the corresponding skill.

**Do not:** Load skills by default, or load multiple skills “to cover everything.” Pick at most the one skill that fits the current case or the user’s mention.

---

## All skills (from list) grouped by use case

### 1. React (components, hooks, TypeScript, React 19)

| Skill | Path | Use when |
|-------|------|----------|
| react-typescript | `.cursor/skills/react-typescript/SKILL.md` | Typed components, hooks, props, forms, TanStack Query in UI, error boundaries |
| react-dev | `.cursor/skills/react-dev/SKILL.md` | React TypeScript, event typing, refs, React 19, Server Components |
| react-19 | `.cursor/skills/react-19/SKILL.md` | React Compiler, no manual memo, ref as prop, Actions, Server Components first |
| react-best-practices | `.cursor/skills/react-best-practices/SKILL.md` | General React best practices |
| vercel-react-best-practices | `.cursor/skills/vercel-react-best-practices/SKILL.md` | Vercel-style React best practices |

### 2. Next.js (App Router, RSC, routing, data)

| Skill | Path | Use when |
|-------|------|----------|
| nextjs-app-router-patterns | `.cursor/skills/nextjs-app-router-patterns/SKILL.md` | App Router, RSC, streaming, parallel routes, Server Actions |
| next-best-practices | `.cursor/skills/next-best-practices/SKILL.md` | File conventions, RSC boundaries, async params/headers, directives |

### 3. TanStack Query (server state, cache, mutations)

| Skill | Path | Use when |
|-------|------|----------|
| tanstack-query | `.cursor/skills/tanstack-query/SKILL.md` | TanStack Query v5 setup, useMutationState, optimistic updates, SSR |
| tanstack-query-best-practices | `.cursor/skills/tanstack-query-best-practices/SKILL.md` | Query keys, caching, mutations, error handling, prefetching, SSR |

### 4. State management (client / global state)

| Skill | Path | Use when |
|-------|------|----------|
| zustand-state-management | `.cursor/skills/zustand-state-management/SKILL.md` | Zustand stores, persist, TypeScript, Next.js SSR/hydration |

### 5. Styling, theme, design system

| Skill | Path | Use when |
|-------|------|----------|
| tailwind-v4-shadcn | `.cursor/skills/tailwind-v4-shadcn/SKILL.md` | Tailwind v4 + shadcn setup, @theme, CSS variables, dark mode |
| tailwind-patterns | `.cursor/skills/tailwind-patterns/SKILL.md` | Tailwind usage patterns in components |
| design-system-patterns | `.cursor/skills/design-system-patterns/SKILL.md` | Design tokens, theming, component library architecture |

### 6. Component design and composition

| Skill | Path | Use when |
|-------|------|----------|
| vercel-composition-patterns | `.cursor/skills/vercel-composition-patterns/SKILL.md` | Compound components, avoid boolean props, context providers |
| web-component-design | `.cursor/skills/web-component-design/SKILL.md` | Component design and structure |

### 7. UI/UX and design feature

| Skill | Path | Use when |
|-------|------|----------|
| react-uiux-design-first | `.cursor/skills/react-uiux-design-first/SKILL.md` | Building React components with UX focus, loading/error/empty states, micro-interactions, animations, visual consistency, accessibility |
| web-design-guidelines | `.cursor/skills/web-design-guidelines/SKILL.md` | Review UI, accessibility, design audit |
| feature-design-assistant | `.cursor/skills/feature-design-assistant/SKILL.md` | Feature-level design assistance |

### 8. Frontend structure and conventions

| Skill | Path | Use when |
|-------|------|----------|
| frontend-dev-guidelines | `.cursor/skills/frontend-dev-guidelines/SKILL.md` | File organization, feature structure, data fetching patterns, routing, loading/error, TypeScript standards |

---

## Reminder

Only use a skill when **the case matches** (see “Use when” in each group) or when **the user explicitly mentions or requests** that technology or pattern. Do not invoke skills by default.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yoriichi-Dang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
