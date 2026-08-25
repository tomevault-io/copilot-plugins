## tucaken-app

> Authoritative guide for Codex when working in this repo. Read fully before editing.

# AGENTS.md — tucaken-app

Authoritative guide for Codex when working in this repo. Read fully before editing.

## Stack

- **Runtime**: TanStack Start (SSR) + TanStack Router + TanStack Query + TanStack Form
- **React**: 19 (Server Components aware)
- **Build**: Vite 8, esbuild for server bundle
- **Styling**: Tailwind CSS v4 via `@tailwindcss/vite` — config lives in `src/styles.css` `@theme` block (no `tailwind.config.*`)
- **State**: Zustand (client), TanStack Query (server cache)
- **Validation**: Zod
- **Animation**: `motion` (Motion for React) — import from `motion/react`, never `framer-motion`
- **Auth**: AWS Cognito + `jose`
- **Payments**: Stripe (`@stripe/*`, server `stripe`)
- **Observability**: OpenTelemetry, Pyroscope, Grafana Faro, Pino
- **Tests**: Vitest

Workspace: yarn workspaces. Root + `admin-api/`.

## Package manager — Yarn 4 only

`packageManager: yarn@4.12.0`. Never use `npm`/`pnpm`/`npx`.

- Install: `yarn install`
- Add dep: `yarn add <pkg>` (root) or `yarn workspace admin-api add <pkg>`
- Add dev dep: `yarn add -D <pkg>`
- Run script: `yarn <script>` (not `npm run`)
- Execute binary: `yarn dlx <cmd>` instead of `npx`
- Lockfile `yarn.lock` is committed. Never delete/regenerate without reason.

## Scripts

| Script | Purpose |
|---|---|
| `yarn dev` | Vite dev on port 5001 |
| `yarn build` | Vite build + server bundle |
| `yarn preview` | Preview production build |
| `yarn lint` | ESLint |
| `yarn typecheck` | `tsc --noEmit` |
| `yarn test` | Vitest run |

Before claiming a task done: `yarn typecheck && yarn lint && yarn test`.

## Repo layout

```
src/
  app/                 # TanStack Start flat-file routes (filename dots = segments)
    __root.tsx
    _dashboard.*.tsx   # routes under dashboard layout group
  features/<domain>/   # feature-sliced — primary location for new code
    components/
    hooks/
    api/ or server/
    types.ts
    store.ts           # zustand if needed
  components/
    ui/                # shared primitives (Button, Field, etc.)
    layouts/
    resume/
  contexts/            # cross-feature React contexts only
  hooks/               # cross-feature hooks only
  lib/                 # framework-agnostic utilities, clients, observability
  server/              # server-only code, patches, handlers
  styles.css           # Tailwind v4 @theme tokens
  router.tsx
  routeTree.gen.ts     # generated — never edit
admin-api/             # workspace (separate package)
```

### Where new code goes

- Belongs to a domain (auth, billing, profile, resumes, …) → `src/features/<domain>/`
- Generic primitive used in 2+ features → `src/components/ui/`
- Server-only (Node APIs, secrets, SDKs) → `src/server/` or `src/lib/<area>/server.ts`
- Route file → `src/app/` only. Never put business logic in route files; import from features.

## TanStack — modern patterns

- **Routing**: file-based in `src/app/`. Legacy routes use flat-file convention (dots = segments, e.g. `_dashboard.applications.$slug.tsx`). `_dashboard` = layout group. `$param` = dynamic. `routeTree.gen.ts` is generated — **never** hand-edit.
- **Routing migration (incremental, mandatory for new work)**: repo is migrating flat-file → directory-based. TanStack Router supports both simultaneously; mixing during transition is intended.
  - **New routes**: create directory-based only. `src/app/<segment>/<sub>/route.tsx` (and `index.tsx` for the segment's own page, `$param/route.tsx` for dynamics).
  - **Touching an existing flat route**: if the change is non-trivial (new sibling, rename, splitting logic), migrate that route's whole prefix group to directory form in the same PR. Update all imports; run `yarn typecheck`.
  - **Migration order priority** (high fan-out first): `_dashboard.applications.*` → `_dashboard.resumes.*` → `_dashboard.settings.*` → remaining `_dashboard.*` leaves → top-level routes (`checkout.*`, `sign-in.*`, `github.*`, `articles.preview.*`).
  - **Mirror feature slices**: directory tree under `src/app/_dashboard/<domain>/` should match `src/features/<domain>/`. Route file stays thin — import logic from the feature.
  - **Colocation rule**: route-private helpers go next to `route.tsx` as `-loader.ts`, `-components/`, etc. Shared logic stays in `src/features/<domain>/`.
  - **No new flat-file routes.** Only exception: a one-off leaf with no siblings and no expected growth — document why in PR description.
  - Do not rewrite untouched flat routes en masse; migration is opportunistic, driven by real work.
- **Data**: prefer `createServerFn` for server logic + `queryOptions` for the client. SSR hydration via `@tanstack/react-start` + `@tanstack/react-router-ssr-query`.
- **Loaders**: route `loader` should call `queryClient.ensureQueryData(queryOptions)` so the same query is reused client-side. Avoid raw `fetch` in components.
- **Forms**: `@tanstack/react-form` + `zod-form-adapter`. Schemas live with the feature.
- **Devtools**: `@tanstack/react-router-devtools`, `@tanstack/react-query-devtools` only mounted in dev.
- **Type safety**: rely on generated route types; never cast `as any` to silence the router.

When unsure of **any** library or API in TypeScript code, use the **context7 MCP** (`resolve-library-id` → `query-docs`) **before writing it** — always, not just for `@tanstack/*`. This is mandatory for all TypeScript work and applies to every external package (TanStack, Zod, Hono, Stripe, `ioredis`, `jose`, Motion, AWS SDK, etc.). Prefer context7 over guessing or relying on memory; it prevents wrong-API code and the rework it causes.

## Components — TailwindPlus first, no duplicates

### Source of truth for new components

1. Before creating any UI component, **check TailwindPlus via the `tailwindplus` MCP**:
   - `mcp__tailwindplus__list_component_names` / `search_component_names` to find a match.
   - `mcp__tailwindplus__get_component_by_full_name` with `framework=react`, `tailwind_version=4`, `mode=light` (or `dark`/`system` for app UI; `none` for eCommerce).
2. Port the snippet into the repo:
   - Replace hard-coded palette (`indigo-600`, etc.) with this app's tokens (see Palette below).
   - Keep semantics; rewrite class strings to use existing tokens. Never ship raw TailwindPlus colors.
3. If no TailwindPlus match exists, fall back to Headless UI (`@headlessui/react`) + Heroicons + Lucide icons already in deps. Document why a custom build was needed in the PR.

### Reuse-first rule (no duplicate components)

Before writing a new component:

```
rg -n "export (default )?function <Name>" src/
rg -n "<probable-component-name>" src/components src/features
```

- If a similar component exists and is used: **refactor it for reuse** — extract props, narrow types, rename file/symbol if the new name describes both call sites better. Move it up the tree to `src/components/ui/` if it now serves multiple features.
- Renaming: update all imports in the same commit. Run `yarn typecheck` to catch stragglers.
- Splitting: prefer composition (children/slots, render props, polymorphic `as`) over copy-paste variants.
- Deleting dead duplicates is part of the refactor — do not leave the old file behind.

### Palette / design tokens

- Defined in `src/styles.css` under `@theme`. Add new tokens there, not inline.
- Use Tailwind utility classes that resolve to those tokens. Avoid arbitrary hex (`bg-[#abc]`) outside the theme block.
- Dark mode via `next-themes`. Any new component must render correctly in both modes.

## Animation — Motion for React

Project rule file: `.Codex/rules/motion-react.md` (authoritative; this section summarises).

- Import: `motion/react` (client) or `motion/react-client` (server components). Never `framer-motion`.
- Skill: invoke the `motion` skill for any animation/visibility/transition work.
- MCP: Motion Studio MCP (`motion`) for paid examples, saved transitions, and CSS spring generation. Use `css-spring`, `motion-audit`, `see-transition` skills for tuning.
- Performance:
  - Animate `transform` / `opacity` / `clipPath` / `filter` only on `willChange`.
  - Use independent transforms (`x`, `scaleX`) when composing.
  - Never read `MotionValue.get()` during render — only in effects/`useTransform` callbacks.
- Radix integration: use `asChild` + `motion.<el>`; hoist `open`/`onOpenChange` state for exit anims; `forceMount` on Radix child of `<AnimatePresence>`.

## TypeScript code quality — SonarQube / SonarLint rules

This repo is analysed by **SonarCloud Automatic Analysis** (runs per-PR; the
quality gate blocks the PR on new issues and unreviewed Security Hotspots).
Write new TypeScript to these rules so the gate stays green — they are
requirements, not style preferences. SonarLint surfaces the same rules live in
the IDE.

- **No nested ternaries (`S3358`).** Use `if`/`else`, an early return, or a small
  helper. In JSX, split branches into separate `{cond && <X/>}` expressions or
  extract a render helper — never `a ? … : b ? … : c` in one container.
- **No redundant casts / non-null assertions (`S4325`).** Let the compiler narrow
  via type guards, `typeof`, `instanceof`, and discriminated unions; don't write
  `x as T` or `x!` when the type is already known. Catch errors as `unknown`:
  `catch (e) { if (!(e instanceof Error) || e.name !== 'X') throw e }` — never
  `catch (e: any)`. **Never** `as any` to silence types.
- **No `String(x)` / template coercion of `unknown` or objects (`S6551`).** Guard
  first: `if (typeof x === 'string' && allowed.has(x))`. Any object used in a
  string context must define `toString()`.
- **Optional chaining over `&&` (`S6582`).** `obj?.prop`, `arr?.[0]`, `fn?.(x)` —
  not `obj && obj.prop`.
- **`Number.*` over globals (`S7773`).** `Number.parseInt` / `Number.parseFloat` /
  `Number.isNaN` / `Number.isFinite` — never the bare globals.
- **`Set` for membership checks (`S7776`).** Declare constant allow-lists as
  `new Set([...])` and use `.has()`, not an array + `.includes()`.
- **Stable React keys (`S6479`).** Use a DB id or a stable content string — never
  the array index.
- **No `Math.random()` for ids/tokens (`S2245` — Security Hotspot, fails the
  gate).** Use `crypto.randomUUID()` / `node:crypto`.
- **No `console.*` in app code.** Use the Pino logger (`src/lib/observability`);
  `console` is acceptable only in CLI/ops scripts under `scripts/`.

**Verification discipline (learned the hard way):** fix findings one at a time
and run `yarn typecheck` (plus the touched tests) after each. If removing an
assertion breaks the build, the compiler genuinely needs it — that's a SonarLint
false positive, so keep it. **Never bulk `eslint --fix` type-assertions across
the repo** — it strips load-bearing casts and cascades into hundreds of errors.

## Security — non-negotiable

- **Secrets**: never commit. `.env.local` is local-only. Server-side env access only inside `src/server/` or `src/lib/**/server.ts`. Never reference `process.env.*` in client components.
- **Input validation**: every server boundary (server fn, route loader receiving params, API handler) validates with Zod. Never trust client payloads.
- **HTML**: sanitize any user-provided HTML with `dompurify` before rendering. Avoid raw-HTML React props on unsanitized content.
- **AuthN/Z**: verify JWT with `jose` against Cognito JWKS server-side. Re-check authorization on every server fn — never rely on hidden UI as access control.
- **Stripe**: webhook handlers must verify signature with the raw body. Never log full card data, secrets, or PII. Use idempotency keys for create operations.
- **SQL / queries**: parameterised only. No string-concatenated SQL.
- **Dependencies**: pin via `yarn.lock`. Avoid adding deps for trivial helpers. Audit new deps for maintenance + license before adding.
- **CSP / headers**: set in server handlers; don't disable existing security headers to silence a console warning — fix the offending code.
- **Logging**: use Pino with redaction. Never log tokens, full request bodies, or PII at info level.
- **Error surface**: client errors never leak stack traces or internal IDs.

## Workflow expectations

- Plan before non-trivial work (TanStack route changes, Stripe flows, auth, schema migrations).
- TDD for logic-heavy features (`superpowers:test-driven-development`).
- Tests: colocate (`__tests__` next to source or under `src/__tests__/`). Vitest.
- Commits: follow `git-commit` skill. Never include `Co-Authored-By` trailer.
- Don't edit `routeTree.gen.ts`, `yarn.lock` (regenerate via yarn), or files under `dist/`.

## Quick checks before "done"

```
yarn typecheck
yarn lint
yarn test
```

For UI work: also run `yarn dev`, open the changed feature in browser, exercise golden path + one edge case.

---
> Source: [Nelson-Lamounier/tucaken-app](https://github.com/Nelson-Lamounier/tucaken-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
