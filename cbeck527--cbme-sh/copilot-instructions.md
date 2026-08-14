## cbme-sh

> This is a small Astro 7 static site styled with Tailwind CSS 4 and deployed through

# Repository Guidelines

## Project Structure & Module Organization

This is a small Astro 7 static site styled with Tailwind CSS 4 and deployed through
Cloudflare Wrangler. Route entry points live in `src/pages/`; shared page chrome belongs in
`src/layouts/`, and reusable UI belongs in `src/components/`. Keep mesh-node content and its
derived `MeshNode` type in `src/data/nodes.ts`. Global styles live in `src/styles/global.css`,
while images and local fonts live under `src/images/` and `src/assets/`. Treat `dist/` as
generated output and never edit it directly.

## Build, Test, and Development Commands

- `nix develop`: enter the pinned Node.js and pnpm development environment.
- `pnpm install`: install the exact dependencies from `pnpm-lock.yaml`.
- `pnpm dev`: start Astro's local development server.
- `pnpm check`: check Prettier formatting and run Astro diagnostics; this is the primary
  source check.
- `pnpm fix`: format supported files with Prettier.
- `pnpm build`: produce the static site in `dist/`.
- `pnpm preview`: build and serve the result through Wrangler locally.

Use `pnpm deploy` only for an intentional production deployment.

## Coding Style & Naming Conventions

Prettier and `.prettierrc` are authoritative for formatting. Run `pnpm fix` instead of
manually reformatting files. Name Astro components in `PascalCase` (`MeshNodeCard.astro`),
route files in lowercase, and TypeScript values in `camelCase`. Prefer semantic HTML, typed
props, and straightforward composition over wrappers or abstractions used only once.

## Testing Guidelines

There is currently no unit-test framework or coverage target. Every change must pass
`pnpm check` and `pnpm build`. For visual or navigation changes, exercise all affected routes
at desktop and narrow mobile widths, check for console errors and overflow, and include
before/after screenshots when behavior or appearance changes. Add focused `*.test.ts` tests
if nontrivial standalone logic is introduced.

## Commit & Pull Request Guidelines

Use short, imperative, sentence-case commit subjects, such as `Simplify node card markup`.
Keep commits focused and avoid unrelated cleanup. Pull requests should explain the user-facing
effect, list verification commands, link relevant issues, and include screenshots for visual
changes. Call out Wrangler or dependency changes explicitly.

## Security & Local Assets

Never commit secrets, `.env` files, generated output, or licensed WOFF2 files. Before building,
place the two required TX-02 files described in `src/assets/fonts/README.md` in that directory.

---
> Source: [Cbeck527/cbme.sh](https://github.com/Cbeck527/cbme.sh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
