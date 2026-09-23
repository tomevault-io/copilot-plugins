## tailnext

> Guidance for automated coding agents working in this repository.

# AGENTS.md

Guidance for automated coding agents working in this repository.

## Stack

This project is on **Next.js 16, React 19, Tailwind CSS 4 and TypeScript 6**. A
lot of training data describes older versions, so check before writing code that
looks familiar:

- **Turbopack is the bundler.** Adding a `webpack` key to `next.config.ts` makes
  `next build` fail.
- **`params` and `searchParams` are Promises** and must be awaited. This applies
  in pages, `generateMetadata`, and image generators.
- **`next lint` no longer exists.** Linting runs through `eslint` directly, with
  a flat `eslint.config.mjs`.
- **There is no `tailwind.config.js`.** Tailwind 4 is configured CSS-first, with
  `@theme` in `src/assets/styles/base.css`.
- **`images.qualities` must list every `quality` value used.** Anything not
  declared returns HTTP 400 from the optimizer.

## Version constraints worth knowing

These are not arbitrary; changing them breaks the toolchain:

- **TypeScript stays on 6.x.** `typescript-eslint` declares `<6.1.0`, so
  TypeScript 7 breaks linting.
- **ESLint stays on 9.x.** `eslint-config-next` bundles `eslint-plugin-react`,
  which still caps at `eslint ^9.7` and calls an API ESLint 10 removed.
- **Vitest stays on 4.x.** `@storybook/addon-vitest` peers on `^3 || ^4`.

## Content

Blog posts and legal pages are `.mdx` files under `src/content`, described by
Zod schemas in `content-collections.ts`. Pages read them through
`import { allPosts } from 'content-collections'` — never through the filesystem.
Invalid frontmatter fails the build on purpose.

## Testing

Stories are the test suite. `npm test` renders every story in headless Chromium
and runs axe against it; an accessibility violation fails the run. A new
component needs a story in `src/stories`, or it is untested.

## Before finishing

Run what CI runs:

```bash
npm run lint && npm run typecheck && npm run prettier:check && npm test && npm run build
```

## Conventions

- Import from `~/` (mapped to `src/`) rather than relative paths.
- Page copy lives in `src/shared/data`, typed against `src/shared/types.ts`.
  Widgets render data; they do not hardcode copy.
- Commits follow Conventional Commits; a git hook enforces it.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---
> Source: [arthelokyo/tailnext](https://github.com/arthelokyo/tailnext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
