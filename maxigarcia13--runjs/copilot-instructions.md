## coding-style

> RunJS stack, UI conventions, and coding style (DRY, SOLID, Tailwind, components)


# RunJS — stack & coding style

Before adding dependencies or UI patterns, read `package.json` and match existing code in `src/`.

## Stack

- **React 19** — functional components, hooks, native elements (`<dialog>`, `hidden`, portals)
- **Vite** — build and dev
- **Tailwind CSS v4** — utilities in components; theme tokens in `src/app.css` (`@theme`, CSS variables)
- **Zustand** — global editor/preview state (`src/store/`)
- **Monaco** — code editor
- **`cn`** from `@maxigarcia/js-utils` — class merging (not clsx directly)

## Do not use Framer Motion / Motion

This project has **no** `framer-motion`, `motion`, or similar animation library. Do not add them unless the user explicitly asks.

For UI that shows/hides or moves:

- **Toggle visibility**: `hidden`, conditional render, or native `<dialog>` (see `modal.tsx`, `collapsible-content.tsx`)
- **Simple feedback**: Tailwind utilities (`transition-colors`, `animate-pulse` — see `skeleton.tsx`, `button.tsx`)
- **Positioning**: existing hooks (`use-floating-position`, `use-dismiss`) and `createPortal`
- **Resize / drag**: pointer events and React state (see `resizable-panel.tsx`)

Prefer zero animation over introducing a new library. If animation is required, use Tailwind/CSS only and keep it minimal.

## Don't repeat yourself (DRY)

- Extract shared logic, types, and UI into reusable modules or components.
- Prefer a single source of truth for data, types, and styles.
- If the same pattern appears more than once, refactor it into a shared abstraction.

## Naming

- **Variables and functions**: Use clear, descriptive names that explain intent (e.g. `userDisplayName`, `formatBookingDate`, `isLoading`).
- **Files**: Use **kebab-case** (hyphenated) for new files (e.g. `booking-card.tsx`, `use-booking-form.ts`).

## Components

- Keep components **as small as possible**; one clear responsibility per component.
- **Prefer existing components** before creating new ones; check the codebase for similar UI or logic and reuse or extend it.
- Split large components into smaller subcomponents when they grow beyond a single concern.
- For **React components**, use **named exports** (`export function Button(...)` or `export const Button = ...`), not `export default`.
- When creating wrapper components (e.g. Button, Input), **extend the props** from the underlying HTML element's attributes, e.g. `extends ButtonHTMLAttributes<HTMLButtonElement>`, so the component accepts all native props and forwards them to the DOM.
- One folder per feature: `component.tsx`, `types.ts`, `index.ts`, context files when needed.
- Icons: `src/assets/icons/*.tsx` as small React components.
- Reuse `Button`, `Modal`, `Tooltip`, `Collapsible` before inventing new primitives.
- Accessibility: `aria-*`, semantic HTML, keyboard-friendly patterns already used in modals/tooltips.

## Tailwind component style

- **Borders**: Use **rounded-md** for borders/corners.
- Add `bg-*` and `text-*` **only when necessary**; don't set background or text color on every element. Rely on inheritance and defaults where the design already looks correct.

## SOLID principles

Follow **SOLID** when designing modules, components, and services:

- **S – Single responsibility**: One reason to change per module/component; split if it does more than one thing.
- **O – Open/closed**: Prefer extending behavior (composition, props, wrappers) over modifying existing code.
- **L – Liskov substitution**: Subtypes and implementations should be usable where the base type is expected without breaking behavior.
- **I – Interface segregation**: Prefer narrow, focused interfaces/props over one large contract; don't force consumers to depend on what they don't use.
- **D – Dependency inversion**: Depend on abstractions (interfaces, types, props) rather than concrete implementations; inject dependencies where it helps.

## Clean Architecture

Think in **Clean Architecture** when building features; it complements SOLID.

- **Dependency rule**: Dependencies point **inward**. Core (domain, use cases) must not depend on UI, frameworks, or external services. UI and infrastructure depend on core.
- **Layers** (inner → outer): **Domain / entities** → **Use cases / application** → **Interface adapters** → **Frameworks & drivers** (React, API clients, DB). Keep business logic in `utils` or `services`; keep UI in `components`.
- **Abstractions**: Define ports (interfaces) for external concerns and implement them in adapters. Inject into use cases or components instead of calling concrete APIs from the UI.
- Apply in proportion to the project: small features need only a clear split between UI and services; larger flows benefit from use-case functions and explicit dependencies.

## Package manager and scripts

- Use **npm** to install dependencies and run scripts. Do not use yarn, pnpm, or other package managers unless the user explicitly asks.
- When adding or updating dependencies, **always check and install the latest versions** (e.g. `npm install <package>@latest`).

## When unsure

- If requirements, naming, or structure are unclear, **ask the user** instead of guessing.
- Search `src/components/` for a similar pattern and follow it.
- Do not assume Radix, shadcn, or Motion — they are not part of this codebase.

---
> Source: [MaxiGarcia13/runjs](https://github.com/MaxiGarcia13/runjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
