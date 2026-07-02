## effract

> Write React components as Effect programs. A React Effect Component (REC) is a real React component whose

# effract

Write React components as Effect programs. A React Effect Component (REC) is a real React component whose
body is a generator effract drives **inside React's render pass** — `yield*` an Effect service or
`yield* hook(useState(...))` a real React hook. The same body — and the same `mount` — runs in a SPA,
during SSR, or (resolve-up-front) as an RSC. See [ADR 0001](./docs/adr/0001-fiber-reconciliation.md).

## Task runner

Always use `just`.

```
just verify       # lint + format + typecheck + boundaries + knip + test (the CI gate)
just verify-fix   # same, with lint/format autofixes
just test         # vitest in the library packages
just example bun-ssr        # run an example app
just build-examples         # build all four apps
```

## Architecture

pnpm workspace, single version **catalog** (exact pins, no ranges). Each package is layered hexagonally,
enforced by dependency-cruiser:

```
packages/<name>/src/
  domain/           pure protocol + types — Effect vocabulary, never React
  application/      the interpreter + the server driver — React-free, capabilities injected
  infrastructure/   adapters — the client React binding (react/), the server mount (server/)
  index.client.ts   the `default` export condition (client mount)
  index.server.ts   the `react-server` export condition (server mount)
```

- `@tmonier/effract` — everything: `rec`/`view`/`hook`/signals + `mount`. `mount` has a client and a
  server implementation in one package; the `react-server` export condition (in `package.json`) picks
  which, so a server module and a client module import the identical `mount`. Peers: `react` + `effect`
  only — no Flight/`react-server-dom` dependency.
- `@tmonier/effract-vite` — Vite plugin.
- `apps/shared` — the services + composed `AppLive` layer + components every example renders.
- `examples/` — typechecked call-site recipes.

## Conventions

- React 19.2+ and Effect v4 only. `Console`/Effect for side-effects in library code, not `console`.
- `#domain/*`, `#application/*`, `#infrastructure/*` resolve via each package's package.json `imports`
  field — **not** tsconfig `paths` (which would mis-resolve across packages). Typecheck is one root
  `tsgo` program over all `src` (no project references — tsgo + composite + noEmit conflict).
- Client-only modules (the `rec` client binding, `runtime`, `reactivity`) carry `'use client'`. The
  `rec`/`view` descriptor (`rec-core`) and the server driver are client-free, so a service-only `rec(...)`
  is one value that the same `mount` renders on the client and on the server — no separate server form. A
  body that imports a hook makes its own module client-only; those RECs stay client islands.
- No abusive lint suppressions. Targeted single-line ignores with a rule code and a reason only.
- Tests behavioural and renderer-agnostic where possible — the interpreter is unit-tested with no DOM.

---
> Source: [get-tmonier/effract](https://github.com/get-tmonier/effract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
