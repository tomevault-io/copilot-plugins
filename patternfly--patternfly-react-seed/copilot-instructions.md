## patternfly-react-seed

> This file helps humans and coding agents work safely in this repository. It summarizes how the app is built, how to validate changes, and where code lives.

# Agent instructions (PatternFly React Seed)

This file helps humans and coding agents work safely in this repository. It summarizes how the app is built, how to validate changes, and where code lives.

## What this project is

Official [PatternFly React seed](https://github.com/patternfly/patternfly-react-seed): a small React + TypeScript app with PatternFly layout, React Router, Webpack, and Vitest. It is a reference for wiring PatternFly into a real app, not a component library.

## Stack

- **React** 18, **TypeScript** (strict), **PatternFly** v6 (`@patternfly/react-core`, `@patternfly/react-icons`, `@patternfly/react-styles`)
- **Bundler**: Webpack (`webpack.*.js`); dev server: `webpack-dev-server`
- **Tests**: **Vitest** + Testing Library (`vitest.config.ts`, `src/test/setup.ts`) — not Jest
- **Lint**: ESLint on `./src/` (`.tsx`, `.js`); **unused imports and variables are errors**

## Commands to run before you consider work done

From the repository root, after `npm install`:

| Command | Purpose |
|--------|---------|
| `npm run type-check` | TypeScript (`tsc --noEmit`) |
| `npm run lint` | ESLint on `src/` |
| `npm test` | Vitest unit tests |
| `npm run test:coverage` | Vitest with coverage (requires devDependency `@vitest/coverage-v8`) |
| `npm run ci-checks` | Same as type-check + lint + `test:coverage` (run this before merging when possible) |

Local development:

| Command | Purpose |
|--------|---------|
| `npm run start:dev` | Webpack dev server |
| `npm run build` | Production build into `dist/` |

## Repository layout

- `src/app/` — Application shell: routes, layout, feature pages (Dashboard, Settings, Support, NotFound)
- `src/app/AppLayout/` — Chrome (header/sidebar)
- `src/component/` — Shared or prototype components
- `src/index.tsx` — Entry; dev-only `react-axe` wiring when not in production
- `webpack.common.js` — Shared Webpack config (aliases, loaders)
- `vitest.config.ts` — Test runner and `@app` alias for tests

## Path aliases

- `@app/*` → `src/app/*` (Webpack and Vitest)
- `@assets/*` → PatternFly packaged assets under `node_modules/@patternfly/react-core/dist/styles/assets/*`

Use these for imports instead of long relative paths where the codebase already does.

## PatternFly and UI conventions

- Global styles: import `@patternfly/react-core/dist/styles/base.css` from the app root (see `src/app/index.tsx`).
- Prefer PatternFly React components for structure and layout (`Page`, `PageSection`, navigation patterns) before ad hoc markup or utility-only layouts.
- Match existing file style: quote style, import order (ESLint `sort-imports` is enabled with `ignoreDeclarationSort`).

## Testing

- Tests live next to features (e.g. `src/app/app.test.tsx`).
- Vitest uses `jsdom` and `./src/test/setup.ts` for globals/setup.

## Things that commonly break CI

- **Unused imports or variables** in `src/` — fix or remove them; ESLint treats them as errors.
- **Omitting `npm run lint` or `npm run type-check`** after edits — run at least those before opening a PR.
- **`npm run ci-checks` fails on coverage** — ensure `@vitest/coverage-v8` is installed (`npm install`) and matches the major Vitest version in `package.json`.

## Optional: PatternFly MCP

For up-to-date PatternFly component APIs and docs, agents can use the [PatternFly MCP server](https://www.npmjs.com/package/@patternfly/patternfly-mcp) if the user’s environment has it configured.

## When unsure

- Prefer patterns already used under `src/app/`.
- Read `README.md` for asset loading (`@app`, `@assets`, SVG/bgimages) and script reference.

---
> Source: [patternfly/patternfly-react-seed](https://github.com/patternfly/patternfly-react-seed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
