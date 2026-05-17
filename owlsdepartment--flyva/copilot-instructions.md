## next-package

> Rules for the React/Next.js adapter package


# @flyva/next

## File structure

```
packages/next/
├── index.ts                     # barrel: re-exports common + runtime
├── common/index.ts              # re-exports from @flyva/shared
├── runtime/
│   ├── hooks/                   # React hooks (useFlyva*, useDetachedRoot, useRefStack, useRefState)
│   ├── components/              # FlyvaRoot, FlyvaLink, FlyvaTransitionWrapper
│   └── page-transition-manager/ # 'use client' re-export of shared manager
└── utils/
    └── refReactiveFactory.ts    # bridges useRefState → ReactiveFactory
```

## Conventions

- Every file that uses React hooks or browser APIs must have `'use client';` as the first line
- Import shared code from **`@flyva/shared`** (barrel) or documented subpaths (**`@flyva/shared/page-transition-manager`**, **`@flyva/shared/view-transition`**, …), not relative `../../shared`
- Components use `forwardRef` when they wrap native elements (e.g. FlyvaLink wraps `<Link>`)
- Avoid external runtime deps — no lodash, no classnames. Destructure/inline instead
- `FlyvaRoot` uses a module-level singleton for the manager to survive React re-renders
- `useRefStack` maintains a global `Map` — it is NOT React state. Transitions access it imperatively via `globalGetRefStackItem`

## Peer dependencies

The package peer-depends on `next >=14`, `react >=18`, `react-dom >=18`. Don't import APIs exclusive to a single Next.js version.

---
> Source: [owlsdepartment/flyva](https://github.com/owlsdepartment/flyva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
