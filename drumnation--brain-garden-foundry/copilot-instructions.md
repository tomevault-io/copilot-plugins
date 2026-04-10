## brain-garden-foundry

> Comprehensive React component and application standards


# React Component & Application Standards (Bulletproof‑React Enhanced)

> **Purpose**  Unify front‑end architecture, state, logging, testing, and DX across every app in `/apps/*`, combining our existing guide with proven *Bulletproof React* principles.

---

## 1  Project Layout & Dependency Flow

```text
src/
  app/           # React Router entry (Vite CSR), global providers
  features/      # Self‑contained domain slices (Auth, Billing…)
  components/    # Re‑usable UI primitives (Button, Card…)
  hooks/         # Shared hooks not tied to a feature
  lib/           # Framework‑agnostic utilities (date, math…)
  types/         # Global TypeScript contracts
```

* **Unidirectional flow**  `components → features → app` only (enforced via ESLint `import/no-restricted-paths`).
* **Absolute imports**  Configure `@/*` alias in `tsconfig.json` to eliminate `../../../`.

## 2  Code Quality Guard‑Rails

| Tool                                                 | Purpose                  | Invocation                    |
| ---------------------------------------------------- | ------------------------ | ----------------------------- |
| TypeScript `--strict`                                | Static safety            | `pnpm tsc -p tsconfig.json`   |
| ESLint (+ plugin\:import, plugin:@typescript-eslint) | Style & dependency rules | `pnpm lint`                   |
| Prettier                                             | Auto‑formatting          | `pnpm format`                 |
| Husky + lint‑staged                                  | Pre‑commit gate          | runs `lint`, `format`, `test` |

CI workflow → *fail fast* on lint, type, unit, integration, e2e.

## 3  State Management – Four Buckets

1. **Local component state**   `useState`, `useReducer`.
2. **UI / app state**          Context, Zustand, or Redux Toolkit (keep thin slices).
3. **Server‑cache state**      React Query (preferred) or SWR.
4. **Form state**              React‑Hook‑Form.

> ✱ *Rule:* Never mix buckets—e.g. don’t put form fields in global UI state.

## 4  API & Data Layer

* All HTTP calls live in `lib/api/` with a single axios instance.
* Generate typed hooks via React Query (`useGetUser`). UI remains declarative.
* Centralised error mapping → user‑friendly toasters, log raw error once.

## 5  UI Component Guidelines

### 5.0 Core Principles

* **Co‑location**  `Button.tsx`, `Button.test.tsx`, `Button.stories.tsx` live together.
* **Props first**   Expose minimal surface; favour composition over prop drilling.
* **Styled‑components + Mantine**    Theme tokens only; ban inline hex colours.

### 5.1 File Naming & Barrel Exports

| Artifact      | Convention               | Example                       |
| ------------- | ------------------------ | ----------------------------- |
| Component     | `PascalCase.tsx`         | `UserCard.tsx`                |
| Styles        | `PascalCase.styles.ts`   | `UserCard.styles.ts`          |
| Tests         | `PascalCase.test.tsx`    | `UserCard.test.tsx`           |
| Stories       | `PascalCase.stories.tsx` | `UserCard.stories.tsx`        |
| Hook          | `useCamelCase.ts`        | `useAuth.ts`                  |
| Folder barrel | `index.ts` re‑export     | `export * from './UserCard';` |

*Rules*

* Prefer **named exports** everywhere; avoids refactor pain and improves tree‑shaking.
* Keep a single React component per file unless trivial variations (e.g., sub‑components).
* Never import from a sibling file’s relative path; use the folder barrel + absolute alias (`@/features/auth`).

### 5.2 Shared Components & Atomic Design

> We blend *Atomic Design* with our **components → features → app** dependency flow by treating the `/components` layer as an atomic toolbox.

```
src/components/
  atoms/        # Lowest‑level primitives – Button, Text, Icon
  molecules/    # Groupings of atoms – AvatarWithName, InputWithLabel
  organisms/    # Complex, reusable blocks – Header, Sidebar, DataTable
  layouts/      # Page‑level wrappers – DashboardLayout, AuthLayout
```

* **Dependencies**

  * Atoms 🚫 cannot import molecules or organisms.
  * Molecules may import atoms.
  * Organisms may import atoms and molecules.
  * Layouts may import organisms, molecules, atoms but **never features**.
* **Stories**  Every atom/molecule/organism gets a Storybook story; organisms/layouts also get Playwright interaction tests.
* **Theming**  All atoms rely on Mantine’s theme + styled‑components; higher layers compose without overriding base styles.
* **Promotion rule**  If a feature‑level component is reused twice outside its feature, promote it up the atomic ladder (molecule → organism as complexity grows).

---

## 6  Logging & Observability  (@kit/logger / Pino 8)  Logging & Observability  (@kit/logger / Pino 8)

### Levels & Usage

| Level           | Use‑case                                           |
| --------------- | -------------------------------------------------- |
| `trace`         | Opt‑in request/response body, perf micro‑spans     |
| `debug`         | Branch decisions, config dumps                     |
| `info`          | Lifecycles: server start, page mount, route change |
| `warn`          | Recoverable issues, slow queries (> 500 ms)        |
| `error / fatal` | Unhandled exceptions, failed externals             |

### Instrumentation Rules

1. Import once per environment:

   ```ts
   // Node
   import { log } from '@kit/logger/node';
   // Browser
   import { log } from '@kit/logger/browser';
   ```
2. **Request scoping** (Node) – middleware generates `requestId`, attach via `log.child({ requestId })`.
3. **Structured shape** → `{ msg, scope, path, requestId, userId? … }`.
4. Front‑end timeline: HTTP calls, feature toggles, critical UI state.
5. Back‑end wrappers: `log.timing('db')` around DB/Redis/S3/Plaid.

```ts
export async function fetchUser(id: string) {
  const logger = log.child({ scope: 'fetchUser', userId: id });
  logger.debug({ id }, 'start');
  const user = await db.getUser(id);
  logger.info({ found: !!user }, 'db lookup complete');
  return user;
}
```

> **Deliverables**  ≤ 300 LOC/app, `/kit/logger/examples.ts`, per‑app “Logging & Debugging” README, green CI.

## 7  Error Boundaries & Monitoring

* Wrap every feature entry in `<ErrorBoundary>`; reset on route change.
* Front‑end: capture to Sentry/Logflare; Back‑end: Pino + CloudWatch.

## 8  Testing Strategy

| Scope              | Tooling                        | Focus                       |
| ------------------ | ------------------------------ | --------------------------- |
| Unit / interaction | Vitest + React‑Testing‑Library | Behaviour, not internals    |
| Component visual   | Storybook + Ladle tests        | Snapshot & interaction      |
| Integration        | Vitest                         | Module contracts (API ↔ UI) |
| E2E                | Playwright                     | Full user flows             |

> **Coverage**  Statement ≥ 80 % critical paths; ignore generated code.

## 9  Performance & DX Defaults

* Code‑split via `React.lazy` in `app/` routes.
* Memoise expensive selectors/hooks (`useMemo`, `memo`).
* Fast dev start: `pnpm i` ≤ 90 s, `pnpm dev` hot reload ≤ 300 ms.

## 10  Security & Deployment

* `.env` managed by `@kit/env-loader`; secrets NEVER commit.
* Enforce HTTPS; helmet in Node.
* GitHub Actions: lint → type‑check → tests → build → deploy (Netlify/Fly.io).

---

### Appendix  — TASK ➜ **ADD CONSISTENT LOGGING**

*(verbatim task for implementers)*

```
Context
- Monorepo root `/`; all runnable apps in `/apps/*` (Node & React/Mantine)
- Logger: pino 8 via **@kit/logger**
  • Node  `import { log } from '@kit/logger/node`
  • Browser `import { log } from '@kit/logger/browser`
- Strict TypeScript (ESM); eslint / prettier; `LOG_LEVEL` default **info**

Goals
1. Instrument key events **once** at the right level
   - **info**  lifecycle (server start, route, page mount)
   - **debug** config, branch decisions
   - **warn**  recoverable issues, slow queries > 500 ms
   - **error / fatal** unhandled exceptions, failed externals
   - **trace** opt-in per request (`X-Debug-Trace: 1`)
2. Structured log shape → `{ msg, scope, path, requestId, userId?, … }`
3. Front-end: log network timeline, feature toggles, critical UI state; ignore noisy events.
4. Back-end:
   • requestId middleware → child loggers
   • log req line + status/duration (body only on **debug**)
   • wrap DB / Redis / S3 / Plaid with `log.timing()`
5. Self-log progress via `log.child({ scope: 'LoggingMigration' })`

Deliverables
- Source updated with logging (≤ +300 LOC/app)
- `/kit/logger/examples.ts` (best-practice snippets)
- “Logging & Debugging” README section per app
- CI green (lint, vitest, playwright)

Rules
- **Only** @kit/logger (no `console.log`)
- Skip/adjust files already well logged
- Run `pnpm turbo build test` before commit
```

---

**Adopt these standards from the next commit forward.** Ship fast, refactor confidently, *stay bulletproof*.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drumnation) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
