## agentic-coding-lab

> Fill in your project overview

# AGENT INSTRUCTIONS

## Project Context (Fill in per project)

Fill in your project overview

---

## Stack

React 18, Next.js (App Router), TypeScript (strict), Tailwind CSS, PrimeReact, TanStack Query, Zustand, React Hook Form + Zod, Vitest + RTL, ESLint + Prettier.

## Folder Structure

```
src/
├── app/                          # Next.js App Router pages and layouts
├── components/
│   ├── common/                   # Primitive, domain-agnostic UI atoms
│   │   └── <ComponentName>/
│   │       ├── <ComponentName>.tsx
│   │       └── index.ts
│   ├── shared/                   # Reusable form-fields, data-tables, layout wrappers
│   │   └── form-fields/
│   │       └── <ComponentName>/
│   │           ├── <ComponentName>.tsx   # ONLY the component file — nothing else
│   │           └── index.ts              # barrel export
│   └── modules/                  # Domain-specific module components
│       └── <module>/
│           ├── <ComponentName>.tsx
│           └── index.ts
├── hooks/
│   ├── shared/                   # Hooks reused across modules (e.g. useLookup.ts)
│   └── modules/
│       └── <module>/             # Domain-specific hooks (e.g. useFetchDoctors.ts)
├── test/
│   ├── shared/
│   │   └── <component-name>/     # All test files related to that shared component go here
│   │       └── *.test.ts         # (component tests, utility tests, hook tests — all here)
│   └── modules/
│       └── <module>/             # All test files related to that module go here
│           └── *.test.ts
├── types/
│   ├── shared/                   # Shared types (e.g. lookup.types.ts)
│   └── modules/
│       └── <module>/             # Domain types (e.g. appointment.types.ts)
├── utils/                        # Pure, stateless utility functions (e.g. sanitizeNumericInput.ts)
├── services/                     # API functions — <domain>.api.ts
├── lib/                          # Infrastructure: api-client, api-hooks, utils (cn)
├── providers/                    # React context providers
├── store/                        # Zustand global state slices
└── constants/                    # App-wide constants (ALL_CAPS naming)
```

### Strict File Placement Rules

| Artifact | Location | Example |
|---|---|---|
| Shared component | `components/shared/<category>/<ComponentName>/` | `form-fields/LookupInputField/LookupInputField.tsx` |
| Module component | `components/modules/<module>/` | `modules/appointment/DoctorCard.tsx` |
| Shared hook | `hooks/shared/` | `hooks/shared/useLookup.ts` |
| Module hook | `hooks/modules/<module>/` | `hooks/modules/appointment/useFetchDoctors.ts` |
| Shared types | `types/shared/` | `types/shared/lookup.types.ts` |
| Module types | `types/modules/<module>/` | `types/modules/appointment/appointment.types.ts` |
| Utility function | `utils/` | `utils/sanitizeNumericInput.ts` |
| Shared component test | `test/shared/<component-name>/` | `test/shared/lookup-input/LookupInputField.test.ts` |
| Module component test | `test/modules/<module>/` | `test/modules/appointment/DoctorCard.test.ts` |
| API service | `services/` | `services/lookup.api.ts` |

### Component Folder Contains ONLY
A component folder (e.g. `LookupInputField/`) must contain **only**:
- `<ComponentName>.tsx` — the component itself
- `index.ts` — barrel export

**Never** place the following inside a component folder:
- Test files (→ `test/shared/<component-name>/`)
- Type definitions (→ `types/shared/` or `types/modules/<module>/`)
- Custom hooks (→ `hooks/shared/` or `hooks/modules/<module>/`)
- Utility functions (→ `utils/`)

## Naming

| Thing      | Convention                    |
| ---------- | ----------------------------- |
| Components | `UserCard.tsx` (PascalCase)   |
| Hooks      | `useUserData.ts`              |
| Types      | `userCard.types.ts`           |
| Services   | `userData.api.ts`             |
| Constants  | `MAX_RETRY_COUNT`             |
| Utils      | `formatCurrency` (verb-first) |

## Rules

**TypeScript:** Strict always. No `any` — use `unknown` + narrowing. Props get named interfaces (`UserCardProps`). No `!` or `as` without a comment.

**Components:** Functional only. One per file. Max ~150 lines — split if larger. No inline styles. Use `cn()` for conditional classes.

**useEffect:** Last resort only. Never for data fetching (use TanStack Query) or deriving state (use `useMemo`). Always clean up. Add a comment if used.

**Data:** All server state via TanStack Query. Always handle `isLoading`, `isError`, empty state. API functions in `services/`, consumed via custom hooks.

**Forms:** React Hook Form + Zod always. Use `zodResolver`. Show field-level errors. Form fields live in `components/shared/form-fields/`.

**State:** Server state → TanStack Query. Local UI → `useState`. Global → Zustand/Redux. Never put server data in global store.

**Testing (TDD):** Write tests before/alongside implementation. All test files for a component live in `test/shared/<component-name>/` or `test/modules/<module>/` — never inside the component folder. Test behaviour not implementation. Cover render, interaction, and edge cases.

## Principles

- **SOLID** — single responsibility, open for extension, narrow prop interfaces, depend on abstractions.
- **YAGNI** — don't build for features that don't exist yet.
- **KISS** — simplest correct solution wins. If a junior needs to Google it, simplify it.

## Before Writing Code

Always do the following first:

1. Understand the exact scope of the request.
2. Identify whether the task is:
   - UI only
   - state management
   - API integration
   - form handling
   - validation
   - performance optimization
   - refactor
   - bug fix
3. Check whether an existing shared component, hook, type, utility, or service already solves part of the problem.
4. Prefer extending existing patterns over inventing new ones.
5. List affected files before making multi-file changes.
6. For non-trivial features, briefly state:
   - what will be changed
   - why
   - which files are affected
7. Do not start implementation until the structure is clear.

## Never

- Use `any`, inline styles, or deprecated APIs
- Abuse `useEffect` or skip error/loading states
- Write monolithic components or over-engineered abstractions
- Add TODOs or truncate generated code
- Touch files outside the scope of the request
- Hallucinate library APIs — say so if unsure

## Architecture Boundaries

- Shared components must remain domain-agnostic.
- Module components may depend on shared components, but shared components must not depend on module components.
- Services must not contain UI logic.
- Components must not contain raw API request logic.
- Validation schemas should live close to the form/module unless reused globally.
- Business rules should not live inside JSX.
- Types should be defined near their domain and reused via imports, never duplicated.
- Utilities must stay pure whenever possible.
- Store should not become a dumping ground for unrelated state.

## Next.js App Router Rules

- Default to Server Components unless client interactivity is required.
- Add `"use client"` only when necessary.
- Keep client boundaries as small as possible.
- Do not move everything to client components for convenience.
- Fetch server data on the server whenever practical.
- Use route-level loading and error boundaries where applicable.
- Keep page files thin; push UI into module components.

## Testing Standards

For components and hooks, test:

- render correctness
- user interaction
- edge cases
- loading / error / empty states
- callback behavior
- accessibility-critical behavior where relevant

Do not test:

- internal implementation details
- private helper behavior indirectly covered elsewhere
- third-party library internals

Prefer:

- user-centric queries
- meaningful assertions
- minimal mocking

## Performance Rules

- Do not prematurely optimize.
- Use memoization only when it prevents real re-renders or expensive recalculation.
- Avoid creating unstable inline objects/functions when they cause child rerenders in sensitive trees.
- Virtualize very large lists when needed.
- Split large components when render responsibilities are mixed.
- Avoid unnecessary client components in Next.js App Router.
- Prefer server rendering where interaction is not required.

## Definition of Done

A task is complete only if:

- code compiles logically
- types are correct
- loading/error/empty states are handled
- accessibility basics are covered
- file placement follows project structure
- naming follows conventions
- no dead code remains
- tests are included when relevant
- component API is clean and documented if newly introduced

---
> Source: [k-t18/agentic-coding-lab](https://github.com/k-t18/agentic-coding-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
