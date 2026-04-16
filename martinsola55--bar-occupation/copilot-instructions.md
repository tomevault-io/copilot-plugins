## bar-occupation

> Web application for monitoring and managing bar occupancy in real time.

# Bar Occupation

Web application for monitoring and managing bar occupancy in real time.

## Tech Stack

- **Framework:** Next.js (App Router) with TypeScript
- **Styling:** Tailwind CSS
- **UI:** Custom components with a proprietary design system
- **State:** React hooks + SWR for data fetching
- **Database:** Supabase / PostgreSQL with Prisma

## General Directives

- Always write strict TypeScript (`strict: true`). Never use `any`.
- Functional components with arrow functions. No class components.
- File names in kebab-case. Component names in PascalCase.
- One component per file. Place in `src/components/` or the corresponding feature folder.
- Use `"use client"` only when strictly necessary (events, state hooks, effects).
- Prefer Server Components by default.
- Code and commits in English. Comments only when the logic is not self-evident.

## Skill: senior-frontend

Apply senior frontend development practices at all times:

- Use the scaffolder (`scripts/frontend_scaffolder.py`) to initialize project structures.
- Use the component generator (`scripts/component_generator.py`) to create new components with the correct structure (file, tests, types).
- Run the bundle analyzer (`scripts/bundle_analyzer.py`) before every release to detect heavy or unnecessary dependencies.
- Follow the reference guides:
  - `references/react_patterns.md` for component patterns.
  - `references/nextjs_optimization_guide.md` for Next.js optimization.
  - `references/frontend_best_practices.md` for general conventions.

## Skill: frontend-design

Every interface must have a clear and intentional aesthetic direction:

- **Before coding**, define: purpose, visual tone, audience, and differentiating element.
- **Typography:** Choose distinctive, characterful fonts. Never use Arial, Inter, Roboto, or generic system fonts. Pair a display font with a refined body font.
- **Color:** Use CSS variables for consistency. Palettes with a dominant color and sharp accents. Avoid timid or evenly distributed palettes.
- **Motion:** Prioritize CSS animations. Use Motion (framer-motion) for React when needed. Focus on high-impact moments: page load with staggered reveals, hover states that surprise.
- **Composition:** Asymmetric layouts, overlapping elements, generous negative space or controlled density. Break the grid when it makes sense.
- **Backgrounds & details:** Create atmosphere with gradients, textures, geometric patterns, dramatic shadows. Do not default to flat solid backgrounds.
- **Forbidden:** Generic AI aesthetics (purple gradients on white, predictable layouts, cookie-cutter components).

## Skill: react-best-practices (Vercel)

Apply the 45 Vercel performance rules, prioritized by impact:

### CRITICAL Priority
- **Eliminate waterfalls:** Move `await` into the branches where it is actually used. Use `Promise.all()` for independent operations. Use `Suspense` for content streaming.
- **Optimize bundle:** Import directly, never from barrel files. Use `next/dynamic` for heavy components. Load analytics/logging after hydration. Preload on hover/focus.

### HIGH Priority
- **Server-side:** Use `React.cache()` for per-request deduplication. Minimize data passed to client components. Restructure components to parallelize fetches. Use `after()` for non-blocking operations.

### MEDIUM Priority
- **Client data fetching:** Use SWR for automatic request deduplication.
- **Re-renders:** Do not subscribe to state used only in callbacks. Extract expensive work into memoized components. Use primitive dependencies in effects. Use `startTransition` for non-urgent updates.
- **Rendering:** Use `content-visibility` for long lists. Hoist static JSX outside components. Use ternary, not `&&` for conditionals.

### LOW Priority
- **JS Performance:** Group CSS changes via classes. Use `Map`/`Set` for O(1) lookups. Cache property access in loops. Return early from functions.

## Skill: ui-design-system

Maintain a coherent and scalable design system:

- Use the token generator (`scripts/design_token_generator.py`) to create palettes, typographic scales, and spacing from the brand color.
- Follow the 8pt spacing grid for margins, paddings, and gaps.
- Define design tokens as CSS custom properties in a central file (`tokens.css` or similar).
- Document every component in the system with its variants, states, and props.
- Ensure WCAG 2.1 AA accessibility at minimum: contrast, visible focus, ARIA labels, keyboard navigation.
- Define responsive breakpoints in the token system. Mobile-first.
- Export tokens in the format the project requires (CSS, SCSS, or JSON).

## Project Structure

```
src/
  app/              # Next.js App Router (layouts, pages, API routes)
  components/       # Reusable design system components
  features/         # Domain modules (bar, occupancy, auth, etc.)
  hooks/            # Shared custom hooks
  lib/              # Utilities, clients (supabase, etc.)
  styles/           # Tokens, global variables, theme
  types/            # Shared types and interfaces
```

## Commands

```bash
npm run dev         # Development server
npm run build       # Production build
npm run lint        # Linting with ESLint
npm run test        # Tests
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MartinSola55) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
